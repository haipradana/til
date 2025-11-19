---
title: "Cloudflare Incident"
date: 2025-11-19T17:48:55+07:00
draft: false
tags:
    - Tech

# Author
language: id
---

Banyak banget insiden yang terjadi akhir-akhir ini. Bulan lalu, Oktober 2025, ada AWS Outage, yang diikuti insiden bulan November ini yaitu Cloudflare outage.

## Ribuan website down

Karena cloudflare down. Penyebabnya memang karena ada `.unwrap()` pada code Rust cloudflare. Tapi akar masalahnya ada di produk Bot Managementnya cloudflare. Layanan cloudflare ini down dari sekitar jam 11.30 UTC sampai 14.30. Pertama-tama, ini dikira hyper-scale DDoS attack oleh pihak cloudflare.

Problemnya ada di feature file yang diupdate setiap 5 menit sekali agar Bot Management systemnya up to date dengan threats yang ada. File ini ada limit sizenya, tapi selama ini sizenya hanya 50% dari max size.

Filenya digenerate tiap 5 menit dari ClickHouse database cluster. Sebenernya, dari ClickHousenya itu bisa ngegenerate configuration yang jelek dan bagus. Tapi kalo configuration filenya jelek, bisa otomatis stabilized dalam failing state.

### Bot Management

Bot Management ini adalah sebuah cara untuk ngefilter bot. Neural network dipake untuk ngetrain machine learning modelnya yang outputnya adalah skor bot. Skor botnya ini menentukan bot mana yang dapat mengakses ke site mereka. 

Neural networknya mengambil ***feature configuration file*** sebagai input. Feature file ini digunakan untuk ML dapat memprediksi apakah request ini adalah automation atau real. Feature file direfresh tiap beberapa menit, buat ngedetect bot baru atau jenis attack baru.

Tiba-tiba ada perubahan clickhouse query yang membuat feature file ini sizenya ngedobel, yang akhirnya bikin error. Akhirnya dicoba lagi oleh mereka dengan ngereplace feature filenya manual, it worked but only for a while, karena feature filenya otomatis diupdate lagi setiap beberapa menit.

Di clickhouse cluster mereka itu ada beberapa shard. Shard `r0` dan `default`. Awalnya, query mereka itu hanya di cluster `default`, tapi somehow mereka mau membuatnya menjadi global. Akhirnya, mereka ubah querynya untuk ngeinclude data dari semua shard.

![duplicates](./duplicates.webp)

Dan boom! datanya duplikat semua. Feature filenya naik size dua kali lipat.

Bot Management punya limit machine learning features yang bisa dimasukin sebagai input, diset jadi 200, which is aman banget karena inputnya masih jauh di bawah 200, cuma 60 aja. Nah tapi sejak perubahan query tersebut, inputnya jadi berkali-kali lipat sizenya. Mereka melakukan append features ke dalam sebuah variable dan ada `.unwrap()` disana. Seharusnya, ketika hit limit, itu gausah pake unwrap.

## Rust problem?

Bahasa pemrograman yang digunakan oleh Cloudflare adalah Rust. Tapi bukan berarti ini salah Rust, kecelakaan ini dapat terjadi di semua bahasa pemrograman. Kesalahan ini adalah pure bad practices dari developer Cloudflare, mereka tidak seharusnya menggunakan `.unwrap()` di code mereka. Jika Cloudflare disuruh milih bahasa apa yang paling cocok untuk kasus mereka, yang mana membutuhkan high-throughput, Rust sebenernya pilihan yang tepat. 