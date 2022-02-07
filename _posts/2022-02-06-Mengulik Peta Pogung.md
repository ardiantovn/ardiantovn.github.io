---
title: 'Mengulik Peta Pogung'
author: 'ardiantovn'
layout: posts
date: 2022-02-06
permalink : /posts/Mengulik Peta Pogung
tags:
  - dataviz
---

Banyak sekali orang bilang bahwa memasuki daerah [Pogung](https://www.google.com/maps/search/pogung/@-7.7586339,110.3725765,16z) itu ibarat memasuki [labirin](https://mojok.co/liputan/susul/pogung-labirin-yang-membuat-bingung-dan-teori-belok-kanan/). Salah satu supir ojek daring juga pernah spontan bilang, "Kita memasuki labirin", saat mengantar saya masuk ke Pogung. Saya pun pernah jalan kaki malam hari dan tersesat di Pogung selama 1 jam. Pertanyaannya, kenapa jalanan di Pogung begitu membingungkan banyak orang? 

<p align="center">
  <img src="/assets/pogung/pogung_area.png" alt="Area Pogung" width="300"/>
</p>

Di artikel ini, saya menggunakan pustaka [osmnx](https://github.com/gboeing/osmnx) dan [contoh-contoh kode programnya](https://github.com/gboeing/osmnx-examples/tree/main/notebooks) untuk menggambar peta Pogung dan melihat [karakteristik wilayah](https://geoffboeing.com/publications/osmnx-complex-street-networks/) tersebut. Wilayah berwarna hijau ini merupakan daerah Pogung Dalangan, Pogung Kidul, Pogung Baru, dan Pogung Lor. Idealnya, saya menggunakan semua area hijau itu. Namun, karena keterbatasan yang saya miliki, saya hanya bisa mengambil data di area berwarna merah. Kode program yang saya susun bisa dilihat [di sini](https://github.com/ardiantovn/map_exploration). Saya menggunakan peta mode ```bicycle``` untuk menggambar rute yang bisa dilalui oleh sepeda, mengingat kebanyakan pengguna jalan di wilayah ini jalan kaki, bersepeda, dan naik sepeda motor.

Area Pogung yang diamati memiliki 162 persimpangan, 135 diantaranya adalah pertigaan dan 27 sisanya perempatan. Mengingat rerata panjang jalan adalah 60 meter, itu artinya, pengguna jalan biasanya akan menemukan persimpangan setiap 60 meter perjalanan. Di sisi lain, berdasarkan diagram polar histogram, arah jalan di area Pogung cenderung condong ke timur laut dan barat laut. Barangkali, dua hal ini menyebabkan orang kebingungan ketika berada di Pogung. Pertama, mereka bingung karena banyak menghadapi persimpangan. Kedua, mereka bingung mengenali arah mata angin.

<p align="center">
  <img src="/assets/pogung/pogung_stat.png" alt="Jejaring Jalan di Pogung" width="500"/>
  <img src="/assets/pogung/pogung_orientation.png" alt="Arah Jalan di Pogung" width="300"/>
</p>
 

Peta jejaring jalanan Pogung menunjukkan semua persimpangan yang ada di area tersebut. Simpul-simpul ini diwarnai dengan warna gelap (nilai _betweeness centrality_ rendah) dan warna terang(nilai _betweeness centrality_ tinggi).  Simpul berwarna kuning memiliki nilai _betweeness centrality_ paling tinggi. Sekitar 23% dari semua rute tersingkat yang ada di Pogung melewati simpul kuning ini. Simpul ini banyak menjadi penjembatan rute-rute pendek di Pogung. Di sisi lain, apabila simpul ini ditutup, maka banyak simpul-simpul lain yang akan terputus dan membuat orang berputar otak lagi untuk mencari jalan lain. Simpul-simpul penting lainnya terkonsentrasi di Jalan Pogung Raya dan Jalan Pogung Kidul. Artinya, kedua jalan ini merupakan jalan yang banyak memiliki simpul penjembatan. 

Lalu, bagaimana keluar dari Pogung seandainya kita tersesat dan tidak bisa melihat peta? Menurut saya, karena banyak simpul penting terkonsentrasi di Jalan Pogung Raya dan Jalan Pogung Kidul, maka carilah salah satu dari dua jalan ini. Jika sudah ketemu, mengingat simpul-simpul ini berderet dari arah selatan ke utara, maka putuskan untuk selalu jalan ke utara ataupun ke selatan saja dan abaikan persimpangan yang mengganggu.
