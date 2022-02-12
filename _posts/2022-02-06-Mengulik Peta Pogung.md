---
title: 'Mengulik Peta Pogung'
author: 'ardiantovn'
layout: posts
date: 2022-02-06
permalink : /posts/Mengulik Peta Pogung
tags:
  - dataviz
---

Banyak sekali orang yang bilang kalau mereka memasuki [Pogung](https://www.google.com/maps/search/pogung/@-7.7586339,110.3725765,16z) itu ibarat mereka memasuki [labirin](https://mojok.co/liputan/susul/pogung-labirin-yang-membuat-bingung-dan-teori-belok-kanan/). Salah satu supir ojek daring juga pernah spontan bilang, "Kita memasuki labirin", saat mengantar saya masuk ke Pogung. Saya pun pernah jalan kaki malam hari dan tersesat di Pogung selama 1 jam. Pertanyaannya, kenapa jalanan di Pogung begitu membingungkan banyak orang? 

<p align="center">
  <img src="/assets/pogung/pogung_area.png" alt="Area Pogung" width="300"/>
</p>

Di artikel ini, saya menggunakan pustaka [osmnx](https://github.com/gboeing/osmnx) dan [contoh-contoh kode programnya](https://github.com/gboeing/osmnx-examples/tree/main/notebooks) untuk menggambar peta Pogung dan melihat [karakteristik wilayah](https://geoffboeing.com/publications/osmnx-complex-street-networks/) tersebut. Wilayah berwarna hijau ini merupakan daerah Pogung Dalangan, Pogung Kidul, Pogung Baru, dan Pogung Lor. Idealnya, saya menggunakan semua area hijau itu. Namun, karena keterbatasan yang saya miliki, saya hanya bisa mengambil data di area berwarna merah. Kode program yang saya susun bisa dilihat [di sini](https://github.com/ardiantovn/map_exploration). Saya menggunakan peta mode ```bicycle``` untuk menggambar rute yang bisa dilalui oleh sepeda, mengingat kebanyakan pengguna jalan di wilayah ini adalah pejalan kaki, pesepeda, dan pengendara sepeda motor.

Area Pogung yang diamati memiliki 162 persimpangan, 135 di antaranya adalah pertigaan dan 27 sisanya perempatan. Rerata panjang jalan di Pogung adalah 60 meter, itu artinya pengguna jalan biasanya akan menemukan persimpangan setiap 60 meter perjalanan. Barangkali, banyaknya persimpangan ini jadi penyebab orang bingung. Sementara itu, berdasarkan diagram polar histogram, arah jalan di area Pogung sedikit menyimpang ke arah timur laut dan tenggara serta cenderung memusat ke 4 arah. Dengan demikian, meskipun arah jalannya menyimpang, arah jalan di Pogung relatif mudah dikenali karena penyimpangannya hanya beberapa derajat dan pola sebarannya memusat.

<p align="center">
  <img src="/assets/pogung/pogung_stat.png" alt="Jejaring Jalan di Pogung" width="500"/>
  <img src="/assets/pogung/pogung_orientation.png" alt="Arah Jalan di Pogung" width="300"/>
</p>
 

Peta jejaring jalanan Pogung menunjukkan semua simpul jalan yang ada di area tersebut. Simpul-simpul ini diwarnai menurut nilai _betweeness centrality_. Semakin tinggi nilainya maka semakin terang warna simpulnya.  Simpul berwarna kuning memiliki nilai paling tinggi yakni 23%. Hal itu berarti simpul ini banyak menjadi penjembatan rute-rute pendek di Pogung (sekitar 23% dari semua rute terpendek yang ada di Pogung melewati simpul kuning ini). Apabila simpul ini ditutup, maka banyak rute yang akan terputus dan membuat orang berputar otak lagi untuk mencari jalan lain. Simpul-simpul penting lainnya terkonsentrasi di sepanjang Jalan Pogung Raya dan Jalan Pogung Kidul. Artinya, kedua jalan ini merupakan jalan yang banyak memiliki simpul penjembatan. 

Lalu, bagaimana cara keluar dari Pogung seandainya kita tersesat ? Menurut saya, karena banyak simpul penting dapat ditemui di Jalan Pogung Raya dan Jalan Pogung Kidul, maka carilah salah satu dari dua jalan ini. Jika sudah ketemu, mengingat simpul-simpul ini berderet dari arah selatan ke utara, maka putuskan untuk selalu jalan ke utara ataupun ke selatan saja dan abaikan persimpangan yang mengganggu.
