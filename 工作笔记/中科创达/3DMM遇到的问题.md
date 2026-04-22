---
title: 3DMM遇到的问题
updated: 2023-04-27 07:32:22Z
created: 2023-04-25 02:49:14Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

## 问题一 
问：face3d中2_3dmm.py文件是对拟合过程的一个仿真。通过随机生成的形状和表情参数、设定的pose和缩放参数，然后再投影得到2d关键点。把2d关键点和3d模型进行拟合重新得到参数，完成仿真。
问题：把3D点投影到图像上时，使用了平移和上下翻转操作，如下代码。
但是使用真实的图片，2d关键点是通过检测得到，那么拟合出来的参数，不需要进行平移和上下翻转操作。为什么会有这样的区别？
~~~python
def to_image(vertices, h, w, is_perspective = False):
    ''' change vertices to image coord system
    3d system: XYZ, center(0, 0, 0)
    2d image: x(u), y(v). center(w/2, h/2), flip y-axis. 
    Args:
        vertices: [nver, 3]
        h: height of the rendering
        w : width of the rendering
    Returns:
        projected_vertices: [nver, 3]  
    '''
    image_vertices = vertices.copy()
    if is_perspective:
        # if perspective, the projected vertices are normalized to [-1, 1]. so change it to image size first.
        image_vertices[:,0] = image_vertices[:,0]*w/2
        image_vertices[:,1] = image_vertices[:,1]*h/2
    # move to center of image
    image_vertices[:,0] = image_vertices[:,0] + w/2
    image_vertices[:,1] = image_vertices[:,1] + h/2
    # flip vertices along y-axis.
    image_vertices[:,1] = h - image_vertices[:,1] - 1
    return image_vertices
~~~

答：1. 真实图片检测得到的关键点，其坐标本身就是像素坐标，所以不需要平移操作。
2. 2_3dmm.py中用于拟合的2d点并不是投影到图像上的点，是从仿真的3D点中拿出的68个对应点。
3. 关于上下翻转操作：如果不进行翻转，pitch角会从-180跳到+180度，即抬头低头不是从-90到90，而是在-180度和+180度跳动。

## 问题二
问：参数的拟合，3ddfa的作者实现方法可以得到平移向量中的Z，但是face3d的实现中，Z值都为1。为什么？
答：3ddfa的作者实现方式是参考了论文[Optimum Fiducials under Weak Perspective Projection](../../学习笔记/PaperReading/【MVG】Optimum%20Fiducials%20under%20Weak%20Perspective%20Proj.md)

原本平移向量的Z值是无法确定的，但是3ddfa的作者用了一个近似，把世界坐标系的所有点计算一个平均值mean_X，把相机成像平面的点的z值近似为0（焦距f相对很小），然后做近似计算，如下面的matlab代码。
另外，face3d中的坐标归一化方式比3ddfa的实现更好。
~~~matlab
function [R t s] = WeakPerspective(x, X)
% reference: Optimum Fiducials under Weak Perspective Projection
% by Alfred M.Bruckstein

% center X
mX = mean(X, 2);
X = X - repmat(mX, 1, size(X, 2));

% center x
mx = mean(x, 2);
x = x - repmat(mx, 1, size(x, 2));

% least squares
n = size(x, 2);
A = zeros(n * 2, 8);
for i = 1 : n
    A(2 * i - 1, :) = [X(:, i)', 1,  0, 0, 0, 0];
    A(2 * i, :) = [0, 0, 0, 0, X(:, i)', 1];
end
P = A \ x(:);
P = reshape(P, 4, 2)';
lambda = P(1, 1 : 3);
gamma = P(2, 1 : 3);

% orthonormalization
d1 = sqrt((lambda * lambda') * (gamma * gamma') - (lambda * gamma') * (lambda * gamma'));
d2 = (lambda * lambda') * (gamma * gamma') + norm(lambda) * norm(gamma) * d1 - (lambda * gamma') * (lambda * gamma');
R = zeros(3);
R(1, :) = ((norm(lambda) + norm(gamma)) / (2 * norm(lambda)) + norm(gamma) * (lambda * gamma') * (lambda * gamma') / (2 * norm(lambda) * d2)) * lambda - (lambda * gamma') * gamma / (2 * d1);
R(2, :) = ((norm(lambda) + norm(gamma)) / (2 * norm(gamma)) + norm(lambda) * (lambda * gamma') * (lambda * gamma') / (2 * norm(gamma) * d2)) * gamma - (lambda * gamma') * lambda / (2 * d1);
s = norm(R(1, :)); %alpha in the paper
R(1, :) = R(1, :) / s;
R(2, :) = R(2, :) / s;
R(3, :) = cross(R(1, :), R(2, :));
t = [mx(1); mx(2); 0] - s * R * mX; % 近似计算t_z
end
~~~

## 问题三：数据增强
1. 同一张图的不同数据增强，包括in-plane 和 out-plane旋转、缩放、遮挡等，都不会影响形状参数和表情参数，但是缩放比和pose参数是需要改变的。
2. 图片左右镜像翻转，会改变所有参数。