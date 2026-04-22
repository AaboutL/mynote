---
title: slam基础-MSCKF公式推导
updated: 2023-06-21 09:07:22Z
created: 2022-05-24 01:38:00Z
latitude: 39.91175842
longitude: 116.37922668
altitude: 0.0000
---

# MSCKF 公式推导

<img src="../../../_resources/72de1786cd1bf09473b0c10bd1c8573a.png" alt="72de1786cd1bf09473b0c10bd1c8573a.png" width="833" height="625">
<img src="../../../_resources/b7c1ee306f217f344800eeb62a29c553.png" alt="b7c1ee306f217f344800eeb62a29c553.png" width="832" height="585">
<img src="../../../_resources/b9a84f8d7ce6d5cd30df67cc72cc787a.png" alt="b9a84f8d7ce6d5cd30df67cc72cc787a.png" width="830" height="603">
<img src="../../../_resources/404b2dc955f23e92059b9c1d95b97074.png" alt="404b2dc955f23e92059b9c1d95b97074.png" width="829" height="582">
<img src="../../../_resources/17fdc6cd4fd7f38ab6514160d5e4802b.png" alt="17fdc6cd4fd7f38ab6514160d5e4802b.png" width="829" height="595">
<img src="../../../_resources/6736da2f412007339e29e24c2a90ccd6.png" alt="6736da2f412007339e29e24c2a90ccd6.png" width="827" height="579">
<img src="../../../_resources/039c3a2b1995d9aca633926fe8e4b9b3.png" alt="039c3a2b1995d9aca633926fe8e4b9b3.png" width="826" height="596">
<img src="../../../_resources/22f743bc2b0719fe0aac1c80fc48e3e0.png" alt="22f743bc2b0719fe0aac1c80fc48e3e0.png" width="826" height="595">