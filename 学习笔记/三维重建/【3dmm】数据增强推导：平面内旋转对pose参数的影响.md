---
title: 【3dmm】数据增强推导：平面内旋转对pose参数的影响
updated: 2023-07-13 07:47:26Z
created: 2023-05-15 11:48:44Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

# 包含转换到图像平面的旋转和平移
数据原始参数为：$f, R_{3d}, t_{3d}, sp, ep$
正交投影矩阵为：$P_{j2 \times 3}$
则world系中的3D顶点坐标为：$V = mu + w_shp * sp + w_exp * ep$
camera系中的3D顶点坐标为：$V_{3d} = fR_{3d}V + t_{3d}$
正交投影的2D点：$V_{2d} = P_{2 \times 3}V_{3d}$
转换到图像平面：$P_{2d} = R_{-1}V_{2d} +t_c, 其中，R_{-1}=[[1, 0][0, -1]], t_c=[0, h+1]$

假设平面内的旋转平移为：$R_{2d}, t_{2d}$

$$
\begin{aligned} 
p_{2d-aug} &= R_{2d}P_{2d} +t_{2d} \\
&=R_{2d}(R_{-1}V_{2d}+t_c) + t_{2d} \\
&=R_{2d}R_{-1}V_{2d} + R_{2d}t_c +t_{2d} \\
&=R_{2d}R_{-1}P_{j2 \times 3}V_{3d} + R_{2d}t_c +t_{2d} \\
&=fR_{2d}R_{-1}P_{j2 \times 3}R_{3d}V + R_{2d}R_{-1}P_{j2 \times 3}t_{3d} + R_{2d}t_c + t_{2d} \tag{1}
\end{aligned} 
$$

已知如下：

$$
P_{2d-aug} = R_{-1}V_{2d-aug} + t_c \\ 
\Rightarrow V_{2d-aug} = R_{-1}^{T}(P_{2d-aug} - t_c) \tag{2}

$$

把式（1）带入式（2）：

$$
\begin{aligned} 
V_{2d-aug} & = R_{-1}^T(fR_{2d}R_{-1}P_{j2 \times 3}R_{3d}V + R_{2d}R_{-1}P_{j2 \times 3}t_{3d} + R_{2d}t_c + t_{2d} - tc) \\ 
&= fR_{-1}^TR_{2d}R_{-1}P_{j2 \times 3}R_{3d}V + R_{-1}^TR_{2d}R_{-1}P_{j2 \times 3}t_{3d} + R_{-1}^TR_{2d}t_c + R_{-1}^T(t_{2d} - tc) \tag{3}
\end{aligned} 
$$

所以，增强之后，sp，ep不变

$$
\begin{aligned}
f &= f \\
R_{aug} &= R_{-1}^TR_{2d}R_{-1}P_{j2 \times 3}R_{3d} \\
t_{aug} &= R_{-1}^TR_{2d}R_{-1}P_{j2 \times 3}t_{3d} + R_{-1}^TR_{2d}t_c + R_{-1}^T(t_{2d} - tc)
\end{aligned} \tag{4}
$$

# 不包含转换到图像平面的旋转和平移
上面的过程中，人脸图片是正的，关键点也是正的。但是3dmm拟合人脸参数的时候，使用的关键点做了一个上下翻转，所以使用3dmm拟合出来的参数恢复像素点之后，还需要对恢复之后的点做一个上下翻转操作：$y = img.h - y + 1$。这个上下翻转的操作被融入到了上面的过程，即：转换到图像平面：$P_{2d} = R_{-1}V_{2d} +t_c, 其中，R_{-1}=[[1, 0][0, -1]], t_c=[0, h+1]$。
这样就导致一个问题：增强的参数中t包含原图的h信息，用增强后的参数恢复的像素点，其y坐标是相对原图的，而不是crop之后的图。这样训练出来的模型，用其推理出来的参数恢复关键点时，在上下翻转像素点时，需要用h=450（300W-LP图像的尺寸）来减y坐标，即$y = 450 - y + 1$。如果原图的尺寸不一致，就导致训练参数不一致。
所以，需要消除原图的影响，那么就不能把上下翻转的操作引入。
## 具体方法
由于拟合3dmm参数时，使用的关键点是上下翻转的，所以数据增强时使用的数据也要用翻转的，包括：
1. 上下翻转的原图
2. 上下翻转的相对原图的关键点
3. 上下翻转的关键点的boundingbox

把上面公式中的$R_{-1}, t_c$去掉后，得到的$V_{2d-aug}$ 如下：
$$
V_{2d-aug} = fR_{2d}P_{j2 \times 3}R_{3d}V + R_{2d}P_{j2 \times 3}t_{3d} +  t_{2d} \tag{5}
$$
得到增前后的R、t为：
$$
\begin{aligned}
f &= f \\
R_{aug} &= R_{2d}P_{j2 \times 3}R_{3d} \\
t_{aug} &= R_{2d}P_{j2 \times 3}t_{3d} + t_{2d}
\end{aligned} \tag{4}
$$
上面的$R_{2d}$中包含缩放s，在代码中，需要把缩放s分离出来，乘到f中去。
代码如下：

~~~python
import os
import numpy as np
import scipy.io as sio
import cv2
from math import cos, sin


def similarity_transform_with_R_transpose(vertices, s, R, t3d):
    t3d = np.squeeze(np.array(t3d, dtype = np.float32))
    transformed_vertices = s * vertices.dot(R) + t3d[np.newaxis, :]
    return transformed_vertices


def similarity_transform(vertices, s, R, t3d):
    t3d = np.squeeze(np.array(t3d, dtype = np.float32))
    transformed_vertices = s * vertices.dot(R.T) + t3d[np.newaxis, :]
    return transformed_vertices


def to_image(vertices, h, w, is_perspective = False):
    image_vertices = vertices.copy()
    if is_perspective:
        # if perspective, the projected vertices are normalized to [-1, 1]. so change it to image size first.
        image_vertices[:,0] = image_vertices[:,0]*w/2
        image_vertices[:,1] = image_vertices[:,1]*h/2
    R = np.array([[1, 0, 0], [0, -1, 0], [0, 0, 1]])
    t = np.array([[0, h + 1, 0]])
    image_vertices = image_vertices.dot(R.T) + t
    return image_vertices.astype(np.float32)


def make_square(box):
    w = abs(box[2] - box[0])
    h = abs(box[3] - box[1])
    big_w_flag = w > h
    diff = abs(w - h)
    box_new = box.copy()
    if big_w_flag:
        box_new[1] = box[1] - diff // 2
        box_new[3] = box_new[1] + w
    else:
        box_new[0] = box[0] - diff // 2
        box_new[2] = box_new[0] + h
    return box_new


def data_aug(box):
    w = box[2] - box[0]
    h = box[3] - box[1]
    assert w == h, "Bbox is not a square!"

    # 1. box size aug
    delta_s = w // 10
    j_x1 = np.random.randint(-delta_s, delta_s)
    j_y1 = np.random.randint(-delta_s, delta_s)
    j_x2 = np.random.randint(-delta_s, delta_s)
    j_y2 = np.random.randint(-delta_s, delta_s)
    box_new = [box[0] + j_x1, box[1] + j_y1, box[2] + j_x2, box[3] + j_y2]
    box_vertex = np.array([[box_new[0], box_new[1]], 
                           [box_new[0], box_new[3]], 
                           [box_new[2], box_new[3]]], dtype=np.float32)

    # 2. box angle aug
    center = [(box_new[0] + box_new[2]) // 2, (box_new[1] + box_new[3]) // 2]
    j_angle = np.random.normal(0, 50)
    R = cv2.getRotationMatrix2D(center, j_angle, 1).astype(np.float32)
    return box_vertex, R


def process_with_R_transpose(img, pts, box, s, R, t, dst_size, aug=True):
    ## The output matrix is the a matrix. The dot production of the vertics and matrix is : V(Nx3) * M(3x3)
    h, w, c = img.shape
    box = make_square(box)
    if aug:
        ori_pts, R_t1 = data_aug(box)
    else:
        ori_pts = np.array([[box[0], box[1]], 
                            [box[0], box[3]], 
                            [box[2], box[3]]], dtype=np.float32)
        R_t1 = np.array([[1, 0, 0], [0, 1, 0]], dtype=np.float32)
    dst_pts = np.array([[0, 0], [0, dst_size-1], [dst_size-1, dst_size-1]], dtype=np.float32)
    R_t = cv2.getAffineTransform(ori_pts, dst_pts)
    R_2 = R_t[0:2, 0:2].dot(R_t1[0:2, 0:2])
    t2d = R_t[0:2, 0:2].dot(R_t1[:, 2]) + R_t[:, 2]
    R_t = np.hstack([R_2, t2d[:, np.newaxis]])
    t2d = np.append(t2d, 0)

    R_aug = np.eye(3, 3)
    R_aug_s = np.eye(3, 3)
    R_aug[0:2, 0:2] = R_t[0:2, 0:2]
    det = np.linalg.det(R_t[0:2, 0:2])
    R_aug_s[0:2, 0:2] = R_t[0:2, 0:2] / det

    img_aug = cv2.warpAffine(img, R_t, (dst_size, dst_size))
    # rotate pts_gt
    one = np.ones([1, pts.shape[0]])
    pts_homo = np.vstack((pts.T, one))
    pts_aug = R_t.dot(pts_homo)

    # rotate the pose para: s R t
    R_tmp_s = R_aug_s.T
    R_tmp = R_aug.T
    R_new = R.T.dot(R_tmp_s)
    t_new = R_tmp.dot(t) + t2d
    s_new = s * det
    return img_aug, pts_aug, R_new, t_new, s_new


def process(img, pts, box, s, R, t, dst_size, aug=True):
    h, w, c = img.shape
    box = make_square(box)
    if aug:
        ori_pts, R_t1 = data_aug(box)
    else:
        ori_pts = np.array([[box[0], box[1]], 
                            [box[0], box[3]], 
                            [box[2], box[3]]], dtype=np.float32)
        R_t1 = np.array([[1, 0, 0], [0, 1, 0]], dtype=np.float32)
    dst_pts = np.array([[0, 0], [0, dst_size-1], [dst_size-1, dst_size-1]], dtype=np.float32)
    R_t = cv2.getAffineTransform(ori_pts, dst_pts)
    R_2 = R_t[0:2, 0:2].dot(R_t1[0:2, 0:2])
    t2d = R_t[0:2, 0:2].dot(R_t1[:, 2]) + R_t[:, 2]
    R_t = np.hstack([R_2, t2d[:, np.newaxis]])
    t2d = np.append(t2d, 0)

    R_aug = np.eye(3, 3)
    R_aug_s = np.eye(3, 3)
    R_aug[0:2, 0:2] = R_t[0:2, 0:2]
    det = np.linalg.det(R_t[0:2, 0:2])
    R_aug_s[0:2, 0:2] = R_t[0:2, 0:2] / det

    img_aug = cv2.warpAffine(img, R_t, (dst_size, dst_size))

    # rotate pts_gt
    one = np.ones([1, pts.shape[0]])
    pts_homo = np.vstack((pts.T, one))
    pts_aug = R_t.dot(pts_homo)

    # rotate the pose para: s R t
    R_tmp_s = R_aug_s
    R_tmp = R_aug
    R_new = R_tmp_s.dot(R)
    t_new = R_tmp.dot(t) + t2d
    s_new = s * det
    return img_aug, pts_aug, R_new, t_new, s_new
~~~