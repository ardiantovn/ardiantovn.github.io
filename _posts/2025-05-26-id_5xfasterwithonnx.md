---
title: 'Jalankan Model Deep Learning 4~5x Lebih Cepat dengan ONNX Runtime'
author: 'ardiantovn'
layout: posts
date: 2025-05-26
permalink : /posts/Jalankan Model Deep Learning 4~5x Lebih Cepat dengan ONNX Runtime
tags:
  - riset
  - deep learning
---

[[Read the English version here](https://medium.com/@ardiantovn/run-the-deep-learning-model-4-5x-faster-with-onnx-runtime-760db96cabdb)]
## Apa itu ONNX?

Setelah melatih model prediksi beam berbasis GPS, saya menjadi tertarik untuk mengoptimalkan model deep learning saya untuk inferensi yang lebih cepat. Saya membaca beberapa teknik, termasuk [quantization](https://huggingface.co/docs/optimum/concept_guides/quantization) dan [menjalankan model dengan Open Neural Network eXchange (ONNX) Runtime](https://youtu.be/M4o4YRVba4o). Namun, saya lebih tertarik dengan ONNX karena ini adalah alat yang menawarkan interoperabilitas (misalnya, melatih model di Python dan men-*deploy*-nya di Rust) dan inferensi yang lebih cepat. Selain itu, penggunaannya mudah karena saya hanya perlu mengekspor model yang telah dilatih ke format ONNX dan memuatnya melalui ONNX Runtime. Di sini, saya ingin berbagi pengalaman saya menggunakan sebagian kode saya di [repo ini](https://github.com/ardiantovn/gpsbeam) tentang cara menggunakan ONNX Runtime dan seberapa cepatnya.

## Bagaimana cara menggunakan ONNX Runtime?

Misalkan, saya telah melatih model di PyTorch. Hal pertama yang perlu saya lakukan adalah menyimpannya ke format ONNX. Untuk menyimpan format ini, saya perlu menyiapkan contoh input model dan berikut adalah contoh data input saya:

```
>> test_dataset[0]

{'input_from_scenario': tensor([ 0,  0,  0,  0, 23, 23, 23, 23]),
 'input_seq_idx': tensor([0, 0, 0, 0, 1, 1, 1, 1]),
 'input_speed': tensor([0.0000, 0.0000, 0.0000, 0.0000, 7.6056, 6.4871, 5.8160, 4.6976]),
 'input_height': tensor([  0.0000,   0.0000,   0.0000,   0.0000, 103.6745, 103.6745, 103.6745,
         103.6745]),
 'input_value': tensor([[ 0.0000,  0.0000,  0.0000,  0.0000],
         [ 0.0000,  0.0000,  0.0000,  0.0000],
         [ 0.0000,  0.0000,  0.0000,  0.0000],
         [ 0.0000,  0.0000,  0.0000,  0.0000],
         [-0.3218, -0.7464,  0.5825,  4.6509],
         [-0.3247, -0.7451,  0.5826,  4.6509],
         [-0.3270, -0.7440,  0.5827,  4.6509],
         [-0.3297, -0.7427,  0.5829,  4.6509]]),
 'input_beam_idx': tensor([ 0,  0,  0,  0, 14, 14, 14, 14]),
 'input_pitch': tensor([ 0.0000,  0.0000,  0.0000,  0.0000, -3.4000, -3.3000, -2.8000, -1.4000]),
 'input_roll': tensor([ 0.0000,  0.0000,  0.0000,  0.0000, 13.1000, 12.6000, 12.2000,  9.5000]),
 'label': tensor([14, 14, 14, 14]),
 'future_sample_idx': tensor([206, 207, 208, 209]),
 'input_sample_idx': tensor([  0,   0,   0,   0, 203, 204, 205, 206])}
```

Di sini, saya memiliki sebuah set data tensor yang disebut `test_dataset`. Set data ini memiliki beberapa sampel dan setiap sampel berisi kamus. Masukan model adalah tensor di dalam kunci bernama `input_value`. Sekarang, saya akan menyimpan model saya ke format ONNX.

```
# pindahkan input sampel dan model ke CPU
sample_input = next(iter(self.test_loader))['input_value']
sample_input_cpu = sample_input.cpu()
model_cpu = self.model.cpu()

torch.onnx.export(model_cpu, 
                sample_input_cpu, 
                self.onnx_model_fname,
                export_params=True,
                do_constant_folding=True,
                opset_version=10,
                input_names=['input'],
                output_names=['output'],
                dynamic_axes={'input': {0: 'batch_size'},
                            'output': {0: 'batch_size'}})
```

Setelah memindahkan `sample_input` dan `model` yang telah dilatih ke CPU, saya hanya perlu menjalankan `torch.onnx.export()` dan menamainya sesuai dengan yang dideklarasikan dalam variabel `self.onnx_model_fname`. Saya mengekspor bobot model dengan mengatur `export_params=True`. Saya menggunakan [optimasi *constant folding*](https://www.geeksforgeeks.org/constant-folding/) yang mengoptimalkan model dengan menghitung terlebih dahulu operasi yang hanya melibatkan nilai konstan dan menggantinya dengan hasilnya. Variabel `opset_version` menunjukkan [versi ONNX yang ditargetkan](https://onnx.ai/sklearn-onnx/auto_tutorial/plot_cbegin_opset.html). Saya memberikan nama untuk node input dan keluaran melalui `input_names` dan `output_names`. Karena saya ingin mengaktifkan inferensi *batch* dinamis, yang berarti ukuran *batch* mungkin berbeda setiap kali saya melakukan inferensi, saya mengatur `dynamic_axes` dengan kamus. Kamus ini berisi kunci untuk node input dan keluaran, dan saya menentukan bahwa dimensi pertama tensor mewakili ukuran *batch*.

Sekarang, untuk melakukan inferensi model menggunakan ONNX Runtime, saya perlu mengimpor library `onnxruntime` dan model ONNX yang telah disimpan. Saya mengatur [execution provider](https://onnxruntime.ai/docs/execution-providers/) sebagai `"CPUExecutionProviders"` karena saya ingin menjalankan inferensi di perangkat keras CPU.

```
import onnxruntime as ort

onnx_fname = r'./arch_cnn-ed-gru-model_nclass_32_.onnx'
ort_session = ort.InferenceSession(onnx_fname,
                                   providers=["CPUExecutionProvider"])
```

Kemudian, saya memuat sampel baru dari set data tensor, mengubah bentuknya untuk menambahkan dimensi *batch*, dan menyusun kamus sebagai input ONNX. Baris `ort_session.get_inputs()[0].name` sama dengan menuliskan secara eksplisit `input` (berasal dari nama node input). Saya menjalankan fungsi yang telah didefinisikan sebelumnya `to_numpy()` untuk mengkonversi sampel dari tensor ke array numpy (lihat repo). Inferensi dilakukan dengan `ort_session.run(['output'], ort_inputs)`. Keluaran dari model ONNX adalah daftar nilai probabilitas, oleh karena itu saya menggunakan fungsi `softmax()` untuk mengubah skala rentang nilai keluaran dan mengurutkannya untuk mendapatkan 5 prediksi teratas.

```
# dapatkan sampel dari dataset tensor
sample = test_dataset[0]['input_value']

# Ubah bentuk input untuk model ONNX (tambahkan dimensi batch)
input_reshaped = sample.unsqueeze(0)

# Jalankan inferensi ONNX
# ini mirip dengan membuat kamus {'input': tensor()}
ort_inputs = {ort_session.get_inputs()[0].name: to_numpy(input_reshaped)}
ort_outs = ort_session.run(['output'], ort_inputs)

# Dapatkan probabilitas dari logits
all_pred_steps = ort_outs[0][0] # 4 langkah prediksi

# Terapkan softmax ke semua prediksi sekaligus
probabilities_all = softmax(all_pred_steps)
# Dapatkan 5 indeks teratas untuk semua prediksi sekaligus
top_5_beams_all = np.argsort(probabilities_all, axis=1)[:, ::-1][:, :5]
```

Berikut adalah contoh keluaran ONNX. Untuk memberikan konteks, saya membuat model yang memproses masukan berupa data GPS dan mengembalikan mengembalikan 4 prediksi (beam saat ini dan tiga index beam selanjutnya). Setiap prediksi adalah daftar dari 5 rekomendasi beam terbaik.

```
ONNX top 5 beam indices: [[14 15 13 16 19]
 [14 15 13 16 19]
 [14 15 13 16 19]
 [14 13 15 16 19]]
```

## Seberapa Cepat ONNX?

Saya mengukur model Torch dan ONNX menggunakan pustaka `timeit`.

```
num_iteration = 10_000
# Ukur waktu inferensi ONNX
onnx_time_ms = timeit.timeit(onnx_inference, number=num_iteration) * 1000 / num_iteration

# Ukur waktu inferensi PyTorch
torch_time_ms = timeit.timeit(torch_inference, number=num_iteration) * 1000 / num_iteration
```

Dan berikut adalah hasilnya:

```
Waktu inferensi ONNX: 0.2674 ms
Waktu inferensi PyTorch: 1.2307 ms
ONNX 4.60x lebih cepat dari PyTorch
```

Hasil benchmark menunjukkan bahwa model ONNX melakukan inferensi sekitar 4,6 kali lebih cepat daripada model PyTorch. Meskipun mungkin ada sedikit variasi dalam kinerja antar pengujian, saya secara konsisten mengamati peningkatan kecepatan dalam kisaran 4~5x ketika menggunakan ONNX runtime. Inferensi yang lebih cepat ini disebabkan oleh beberapa optimasi grafik (misalnya optimasi *constant folding*), seperti yang dijelaskan di [situs ONNX](https://onnxruntime.ai/docs/performance/model-optimizations/graph-optimizations.html#graph-optimization-levels). 