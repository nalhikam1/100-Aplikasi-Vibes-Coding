Meningkatkan Model Bahasa dengan Unsloth: Memanfaatkan LoRA, QLoRA, dan Adaptor untuk Aplikasi Dunia Nyata
Meningkatkan Model Bahasa dengan Unsloth: Memanfaatkan LoRA, QLoRA, dan Adaptor untuk Aplikasi Dunia Nyata

SHUBHAM SHANKARSHUBHAM SHANKAR
SHUBHAM SHANKAR
Diterbitkan 5 Nov 2024
+ Ikuti
Membuka potensi model bahasa besar melalui adaptasi yang efisien.
Dalam lanskap dinamis Pemrosesan Bahasa Alami (NLP), Model Bahasa Besar (LLM) mengubah industri, memicu obsesi di antara perusahaan yang ingin membuka potensi mereka. Sementara LLM unggul dalam memahami dan menghasilkan bahasa manusia, menyempurnakan model ini menjadi penting untuk menghadirkan aplikasi yang disesuaikan dengan kinerja tinggi.

Saat organisasi mencari solusi yang disesuaikan untuk tantangan tertentu, permintaan akan model yang disesuaikan terus melonjak. Melepaskan kemalasan — kerangka kerja sederhana open source yang dirancang untuk penyempurnaan LLM yang efisien dan tepat. Dengan menurunkan semua langkah matematika yang berat komputasi dan kernel GPU tulisan tangan secara manual, Unsloth dapat secara ajaib mempercepat pelatihan tanpa perubahan perangkat keras apa pun. Pendekatan inovatif ini memungkinkan kami untuk berhasil menyesuaikan model dan menerapkan varian yang disesuaikan, membuatnya dapat diakses untuk berbagai aplikasi.

Di blog ini, saya mengeksplorasi perjalanan penyempurnaan saya dengan Unsloth, menyoroti manfaat dari pendekatan ini dan merinci proses penerapan model GGUF yang disempurnakan ke Hugging Face. Dengan menggunakan kedua SPenyempurnaan yang disempurnakan (SFT) dan Penyetelan Halus yang Efisien Parameter (PEFT) teknik, saya dapat mengarahkan model ke arah persyaratan khusus melalui pelatihan pada data berlabel, memastikannya dapat menangani aplikasi yang ditargetkan dengan mahir. Sementara itu, PEFT memungkinkan peningkatan kinerja model dengan overhead komputasi minimal, mengoptimalkan efisiensi tanpa mengorbankan kualitas. Bersama-sama, teknik-teknik ini memaksimalkan potensi model dengan cara yang hemat sumber daya.

Lihatlah blog Medium saya.

Penyetelan Halus
Menyempurnakan Model Bahasa Besar (LLM) mengacu pada proses mengadaptasi model yang telah dilatih sebelumnya untuk tampil lebih baik pada tugas atau domain tertentu. LLM awalnya dilatih pada kumpulan data yang luas dari berbagai sumber untuk memahami dan menghasilkan teks mirip manusia. Meskipun model ini bersifat umum, penyempurnaan memungkinkan mereka untuk berspesialisasi dengan melatih kumpulan data khusus tugas yang lebih kecil yang selaras dengan kebutuhan aplikasi atau industri tertentu. Pelatihan tambahan ini memungkinkan model untuk menawarkan output yang lebih relevan, akurat, dan sadar konteks untuk kasus penggunaan tertentu.

Penyempurnaan dapat dilakukan dengan berbagai cara tergantung pada hasil yang diinginkan. Untuk beberapa tugas, hanya beberapa lapisan model yang mungkin memerlukan penyesuaian, sementara di tugas lain, pelatihan ulang yang lebih ekstensif mungkin diperlukan.



Isi artikel
Image Source: Unstructured.
Proses penyempurnaan dapat intensif secara komputasi, tergantung pada ukuran model dan volume data pelatihan, tetapi trade-off adalah model yang berkinerja jauh lebih baik dalam konteks khusus. Selain itu, penyempurnaan memungkinkan peningkatan fleksibilitas dalam aplikasi model, seperti menghasilkan konten kreatif, menerjemahkan bahasa, atau bahkan memberikan nasihat hukum atau medis.

Terlepas dari kelebihannya, penyempurnaan LLM membutuhkan pertimbangan yang cermat untuk menghindari masalah seperti overfitting, di mana model menjadi terlalu terspesialisasi dan kehilangan kemampuan generalisasinya. Selain itu, pertimbangan etis, seperti bias yang ada dalam data pelatihan, harus dikelola untuk memastikan bahwa model yang disesuaikan tetap adil dan akurat.

Namun, jika dilakukan dengan benar, penyetelan halus dapat membuka potensi penuh LLM, menjadikannya alat yang sangat efektif untuk tugas-tugas tertentu sambil mempertahankan kekuatan dan keserbagunaan pelatihan asli mereka yang luas.

Menjelajahi Berbagai Pendekatan untuk Penyempurnaan Model
Saat bekerja dengan model yang telah dilatih sebelumnya, menyempurnakannya agar berkinerja baik pada himpunan data baru sangat penting. Tetapi bagaimana kami mendekati penyempurnaan ini dapat berdampak besar pada kinerja dan efisiensi.



Isi artikel
Image Source: Google Images.
Mari kita pahami strategi paling umum untuk penyempurnaan model dan kapan setiap metode bersinar.

1. Melatih Ulang Semua Parameter

Pendekatan ini melibatkan pelatihan ulang setiap parameter model yang telah dilatih sebelumnya menggunakan himpunan data baru Anda. Meskipun ini bisa menjadi cara yang ampuh untuk menyesuaikan model dengan tugas tertentu, itu harus dibayar — terutama dalam hal sumber daya komputasi. Melatih ulang semua parameter bisa mahal dan memakan waktu. Selain itu, jika himpunan data baru Anda relatif kecil, Anda berisiko overfitting, di mana model menjadi terlalu khusus untuk data baru dan kehilangan kemampuan generalisasinya. Metode ini paling cocok ketika Anda memiliki himpunan data yang besar dan beragam yang memerlukan model untuk mempelajari pola baru dan kompleks yang tidak ada dalam data pelatihan asli.

2. Transfer Pembelajaran

Pembelajaran transfer adalah metode yang lebih efisien dan banyak digunakan yang memanfaatkan pengetahuan yang telah diperoleh model. Alih-alih melatih ulang semua parameter, Anda hanya menyempurnakan lapisan atas atau subset tertentu dari bobot model pada himpunan data baru Anda. Ini sangat berguna ketika himpunan data baru mirip dengan yang asli, karena model dapat mentransfer fitur dan pola yang dipelajarinya ke tugas baru. Pembelajaran transfer secara komputasi lebih ringan daripada melatih ulang semuanya, dan seringkali dapat menghasilkan kinerja yang unggul karena model dimulai dengan fondasi yang kokoh. Ini adalah pilihan yang bagus saat Anda bekerja dengan himpunan data terkait dan ingin menghemat waktu dan sumber daya.

3. Penyetelan Halus yang Efisien Parameter (PEFT)

Seperti namanya, penyempurnaan yang efisien parameter berfokus pada memperbarui hanya sebagian kecil dari parameter model daripada semuanya. Metode ini sangat berharga ketika himpunan data baru Anda kecil atau Anda hanya memerlukan model untuk mempelajari pola tertentu. PEFT meminimalkan jumlah komputasi yang diperlukan, menjadikannya pilihan yang efisien saat Anda membutuhkan hasil yang lebih cepat atau memiliki sumber daya yang terbatas. Meskipun memperbarui parameter yang lebih sedikit, PEFT masih dapat memberikan kinerja yang kuat, terutama dalam skenario di mana model yang telah dilatih sebelumnya sudah memiliki pemahaman yang kuat tentang konteks yang lebih luas.

Metode Penyetelan Halus
Untuk model bahasa besar (LLM), penyempurnaan dapat dilakukan dengan beberapa cara, tergantung pada sifat tugas dan jenis umpan balik yang diterima model.

Mari kita jelajahi tiga metode penyempurnaan yang menonjol: Penyempurnaan yang Diawasi, Pembelajaran Penguatan dengan Umpan Balik Manusia (RLHF)dan Optimasi Preferensi Langsung (DPO).

1. Penyempurnaan yang Diawasi
Apa itu:

Penyempurnaan yang diawasi melibatkan pelatihan model yang telah dilatih sebelumnya pada himpunan data berlabel di mana jawaban yang benar diketahui sebelumnya. Metode ini biasanya menggunakan pasangan input-output (misalnya, prompt dan respons yang diinginkan) untuk mengajarkan model untuk mereplikasi perilaku tertentu atau menghasilkan output yang akurat untuk tugas tertentu.

Cara kerjanya:


Persiapan Data: Himpunan data dikuratori di mana setiap input (pertanyaan, percepatan, dll.) dipasangkan dengan output yang benar (jawaban, tanggapan, dll.). Himpunan data ini sering diberi label secara manual oleh manusia atau dihasilkan melalui metode otomatis (Pembuatan Data Sintetis).
Pelatihan: Model kemudian disetel dengan menyesuaikan parameternya untuk meminimalkan perbedaan antara prediksinya dan output sebenarnya (diukur melalui fungsi kerugian, seperti kehilangan entropi silang).
Obyektif: Tujuannya adalah untuk membuat model lebih akurat dan selaras dengan perilaku spesifik tugas yang diinginkan dengan langsung belajar dari contoh.


Isi artikel
Image Source: Deci AI.
2. Pembelajaran Penguatan dengan Umpan Balik Manusia (RLHF)
Apa itu:

Pembelajaran Penguatan dengan Umpan Balik Manusia (RLHF) adalah metode penyempurnaan yang lebih canggih di mana model dilatih menggunakan umpan balik yang berasal dari evaluasi manusia terhadap responsnya. Model ini tidak hanya belajar dari contoh berlabel tetapi juga dari preferensi atau kritik manusia.

Cara kerjanya:


Evaluasi Manusia: Setelah model menghasilkan output (misalnya, teks), manusia mengevaluasi keluaran ini dan memberikan umpan balik, yang dapat berupa peringkat atau peringkat.
Sinyal Hadiah: Umpan balik manusia diubah menjadi sinyal hadiah, biasanya nilai atau peringkat skalar, yang memberi tahu model seberapa baik kinerjanya.
Pembelajaran Penguatan: Dengan menggunakan sinyal hadiah ini, model menyesuaikan parameternya melalui pembelajaran penguatan, mengoptimalkan responsnya agar lebih selaras dengan harapan manusia.
Tujuannya adalah untuk membuat model lebih selaras dengan nilai, preferensi, dan maksud manusia daripada hanya akurasi statistik.



Isi artikel
Image Source: Serrano.Academy.
3. Optimasi Preferensi Langsung (DPO)
Apa itu:

In 2023 paper “Direct Preference Optimization: Your Language Model is Secretly a Reward Model”, the authors propose a method called Direct Preference Optimization for effectively controlling Large-scale unsupervised Language Models (LLMs) without relying on more complex approaches such as Reinforcement Learning from Human Feedback (RLHF).
Optimasi Preferensi Langsung (DPO) adalah teknik penyempurnaan yang secara langsung belajar dari data preferensi, di mana evaluator manusia memberi peringkat atau memilih di antara output berbeda yang dihasilkan oleh model. Saat menerapkan DPO, model reward tidak lagi diperlukan. Fokusnya adalah pada pengoptimalan model untuk menghasilkan output yang disukai oleh manusia, daripada mengoptimalkan akurasi atau relevansi itu sendiri.

Cara kerjanya:


Pengumpulan Data Preferensi: Model menghasilkan beberapa output kandidat untuk input tertentu, dan evaluator manusia memberi peringkat output ini sesuai dengan yang mereka sukai.
Belajar dari Preferensi: Model ini menggunakan data peringkat ini untuk mempelajari jenis respons mana yang lebih selaras dengan preferensi manusia.
Optimasi: Parameter model disesuaikan untuk memaksimalkan kemungkinan menghasilkan output yang disukai, dengan model belajar untuk memprioritaskan mereka yang cenderung dinilai tinggi oleh evaluator manusia.
DPO bertujuan untuk mengoptimalkan model untuk memilih output yang lebih mungkin memenuhi preferensi pengguna atau untuk menghasilkan respons yang menurut manusia lebih tepat dalam berbagai konteks.



Isi artikel
Image Source: Serrano.Academy.
Teknik Penyempurnaan
Apa itu LoRA?
Lora (Adaptasi Peringkat Rendah) adalah metode yang dirancang untuk menyempurnakan model besar secara efisien dengan memperkenalkan matriks peringkat rendah ke dalam matriks berat model. Tujuan utama LoRA adalah untuk mengurangi biaya komputasi dan penggunaan memori selama proses penyempurnaan tanpa mengorbankan kemampuan model untuk mempelajari informasi khusus tugas. Ini bekerja dengan menambahkan matriks peringkat rendah yang dapat dilatih ke bobot model yang ada daripada memperbarui set lengkap bobot.

Konsep Kunci LoRA:


Perkiraan Peringkat Rendah: Alih-alih melatih semua parameter dalam model, LoRA berfokus pada penambahan matriks peringkat rendah ke pembaruan bobot model. Hal ini membuat proses penyetelan halus lebih efisien dalam memori, karena matriks peringkat rendah membutuhkan lebih sedikit parameter untuk diwakili.
Efisiensi Parameter: LoRA menambahkan parameter tambahan (matriks peringkat rendah) ke model yang ukurannya jauh lebih kecil dibandingkan dengan parameter model aslinya. Idenya adalah bahwa parameter tambahan ini menangkap penyesuaian yang diperlukan untuk tugas baru tanpa perlu memodifikasi seluruh matriks berat.
Membekukan Bobot Pra-Terlatih: LoRA biasanya membekukan sebagian besar bobot yang telah dilatih sebelumnya dan hanya melatih matriks peringkat rendah. Ini mengurangi jumlah parameter yang dapat dilatih dan mempercepat proses pelatihan.
Cara Kerja LoRA

LoRA melakukan dua hal mendasar yang berbeda dari penyempurnaan parameter penuh:


Trek berubah menjadi bobot alih-alih memperbarui bobot secara langsung.
Menguraikan matriks besar perubahan berat menjadi dua matriks yang lebih kecil yang berisi "parameter yang dapat dilatih".
#2 adalah tempat saus rahasia berada.



Isi artikel
Source: Arxiv.
Pertimbangan Praktis

Dalam praktiknya, Peringkat R dari matriks peringkat rendah adalah hiperparameter yang perlu kita sesuaikan. Jika r terlalu kecil, model mungkin tidak beradaptasi dengan baik dengan tugas baru. Jika r terlalu besar, itu dapat meniadakan manfaat efisiensi komputasi LoRA.

Misalnya:


Jika W adalah matriks 1000×1000 dan peringkat r adalah 10, matriks peringkat rendah A dan B masing-masing akan memiliki ukuran 1000×10 dan 10×1000. Ini berarti bahwa alih-alih memperbarui matriks 1000×1000, Anda sekarang memperbarui dua matriks yang lebih kecil, A dan B, masing-masing dengan 10.000 parameter, dengan total 20.000 parameter.
Ini jauh lebih efisien secara komputasi dan memori dibandingkan dengan menyempurnakan semua 1.000.000 parameter secara langsung dalam matriks berat W.

Parameter yang dapat dilatih LoRA sebagai persentase dari total parameter model keseluruhan:



Isi artikel
Image Source: Entry Point AI.
Apa itu QLoRA?
QLoRA dibangun di atas prinsip-prinsip LoRA tetapi terintegrasi Kuantisasi untuk lebih mengurangi jejak memori dan biaya komputasi.



Isi artikel
Image Source: My Notes.
Sementara LoRA berfokus pada pengurangan jumlah parameter dengan menggunakan perkiraan peringkat rendah untuk pembaruan bobot, Kuantisasi mengurangi presisi parameter, semakin mengurangi penggunaan memori.

QLoRA team created a special datatype called a NormalFloat that allows a normal distribution of weights to be compressed from 16-bit floats into 4-bits and then restored back at the end with minimal loss in accuracy.
Hasilnya adalah metode yang memungkinkan model besar untuk disempurnakan menggunakan lebih sedikit perangkat keras, mengurangi persyaratan penyimpanan untuk model dan biaya pelatihan.

Konsep Kunci QLoRA:


Adaptasi Peringkat Rendah (Lora): Seperti yang dijelaskan sebelumnya, LoRA mengurangi jumlah parameter yang dapat dilatih dengan menguraikan pembaruan bobot menjadi matriks peringkat rendah, yang secara efisien menangkap pengetahuan khusus tugas yang diperlukan tanpa mengubah seluruh matriks bobot.
Kuantisasi: Ini adalah proses mengurangi presisi bobot dan aktivasi model. Misalnya, alih-alih menggunakan nilai floating-point 32-bit untuk mewakili parameter, Anda dapat menggunakan bilangan bulat 8-bit atau representasi 4-bit. Ini secara signifikan mengurangi jumlah memori yang diperlukan untuk menyimpan parameter, sehingga memungkinkan pelatihan model yang lebih besar di lingkungan yang dibatasi sumber daya.
QLoRA — LoRA 2.0: Dengan menggabungkan kedua teknik ini, QLoRA menerapkan keduanya dekomposisi peringkat rendah dan Kuantisasi hingga pembaruan berat model. Idenya adalah bahwa matriks peringkat rendah dapat dikuantisasi tanpa secara signifikan mempengaruhi kinerja model, memungkinkan pengurangan besar-besaran dalam penggunaan memori dan overhead komputasi.
Teknik Kuantisasi
Ada berbagai metode kuantisasi yang dapat digunakan dalam QLoRA, seperti:


Kuantisasi Seragam: Ini adalah pendekatan yang paling mudah, di mana nilai dalam matriks diskalakan dan dibulatkan ke bilangan bulat terdekat dalam rentang tetap.
Kuantisasi Tidak Seragam: Metode yang lebih canggih dapat digunakan di mana tingkat kuantisasi tidak seragam melainkan beradaptasi dengan distribusi bobot atau gradien.
Direkomendasikan oleh LinkedIn
Retrieval Augmented Generation for Large Language Models (LLM) 
Retrieval Augmented Generation for Large Language…
Abhijeet Anand  2 tahun yang lalu
Mengapa Model Konsep Besar Adalah Kunci AI yang Benar-benar Cerdas
Mengapa Model Konsep Besar Adalah Kunci AI yang…
Saquib Khan  1 tahun yang lalu
Exploring Retrieval Augmented Generation (RAG) with Large Language Models (LLMs)
Exploring Retrieval Augmented Generation (RAG) with…
Harpreet Sethi  1 tahun yang lalu
Manfaat utama QLoRA adalah bahwa mengukur hanya matriks peringkat rendah (dan bukan parameter model lengkap) membantu mencapai keseimbangan antara efisiensi komputasi dan performa model.



Isi artikel
Image Source: Medium.
Apa itu Adaptor?
Adaptor adalah modul jaringan saraf kecil, biasanya terdiri dari satu atau dua lapisan, yang dimasukkan ke dalam lapisan model yang telah dilatih sebelumnya, seringkali transformator. Adaptor ini dirancang untuk menyempurnakan representasi model untuk tugas atau domain tertentu tanpa memodifikasi bobot seluruh model.

Alih-alih menyempurnakan semua parameter model besar — pendekatan yang mahal secara komputasi dan membutuhkan sejumlah besar data berlabel — adaptor memungkinkan kita untuk hanya melatih lapisan adaptor ringan. Ini secara signifikan mengurangi jumlah parameter yang perlu diperbarui, menjadikan adaptor solusi yang jauh lebih hemat sumber daya untuk adaptasi model.

Dengan mempertahankan pengetahuan dalam model yang telah dilatih sebelumnya dan berfokus pada penyesuaian kecil yang spesifik untuk tugas, adaptor menghemat sumber daya komputasi yang cukup besar dibandingkan dengan melatih ulang seluruh model. Selain itu, beberapa adaptor dapat ditambahkan ke model fondasi yang sama, memungkinkan pembelajaran multi-tugas dan memungkinkan penyesuaian yang mudah di berbagai tugas tanpa gangguan di antara keduanya.



Isi artikel
Image Source: AHead of AI.
Memperkenalkan Unsloth : Pengubah Permainan untuk Model Bahasa Penyempurnaan
Unsloth adalah kerangka kerja canggih yang menyederhanakan dan mengoptimalkan penyempurnaan model bahasa, sehingga memudahkan peneliti dan pengembang untuk meningkatkan kinerja model. Ini terintegrasi secara mulus dengan model sumber terbuka populer yang menawarkan metode penyetelan halus yang fleksibel seperti Lora dan DPO. Opsi ini memungkinkan pengguna untuk menyesuaikan proses penyempurnaan dengan kebutuhan spesifik mereka, meningkatkan kemampuan beradaptasi model untuk berbagai tugas pemrosesan bahasa alami.

Fitur menonjol dari Unsloth adalah penggunaan memorinya yang efisien, yang memungkinkan penyempurnaan pada GPU lokal, membuat tugas NLP tingkat lanjut lebih mudah diakses oleh audiens yang lebih luas. Dengan mengurangi sumber daya komputasi yang diperlukan untuk penyempurnaan, Unsloth menurunkan penghalang masuk, memungkinkan lebih banyak pengguna untuk bereksperimen dan mengoptimalkan model bahasa besar untuk beragam aplikasi.

Menyiapkan Lingkungan untuk Penyempurnaan dengan Colab
Dalam sesi ini, kita akan menyiapkan lingkungan kita di Google Colab, menggunakan GPU tingkat gratis (T5), untuk menyempurnakan model bahasa canggih: Llama3, dikembangkan oleh Meta.

Sebelum kita menyelami proses penyempurnaan, pertama-tama kita akan membahas langkah-langkah untuk menyiapkan pustaka dan konfigurasi yang diperlukan.

Langkah 1: Menginstal Perpustakaan Unsloth

Kita akan mulai dengan menginstal Melepaskan kemalasan library, yang menyediakan alat dan fungsionalitas yang kita butuhkan untuk bekerja dengan model bahasa besar (LLM).

%%capture
# Installs Unsloth, Xformers (Flash Attention) and all other packages!
!pip install "unsloth[colab-new] @ git+https://github.com/unslothai/unsloth.git"

# We have to check which Torch version for Xformers (2.3 -> 0.0.27)
from torch import __version__; from packaging.version import Version as V
xformers = "xformers==0.0.27" if V(__version__) < V("2.4.0") else "xformers"
!pip install --no-deps {xformers} trl peft accelerate bitsandbytes triton        
Langkah 2: Mengimpor Modul yang Diperlukan

Selanjutnya, kita akan mengimpor modul yang diperlukan untuk mengatur dan menyesuaikan model bahasa kita. Ini termasuk Model Bahasa Cepat , yang merupakan komponen kunci dari Unsloth.

from unsloth import FastLanguageModel
import torch        
Langkah 3: Membuat Instans Model

Untuk tugas penyempurnaan kami, kami akan bekerja dengan Llama3 model — model bahasa Meta. Kami akan membuat instance FastLanguageModel dengan konfigurasi tertentu, seperti:


Panjang Urutan Maksimum: Jumlah maksimum token yang dapat diproses model sekaligus.
Tipe Data: Mengonfigurasi presisi (misalnya, float32, float16).
Pemuatan 4-bit: Pengaturan hemat memori yang memungkinkan model berjalan pada spesifikasi perangkat keras yang lebih rendah tanpa mengorbankan banyak kinerja.
max_seq_length = 2048 # Choose any! We auto support RoPE Scaling internally!
dtype = None # None for auto detection. Float16 for Tesla T4, V100, Bfloat16 for Ampere+
load_in_4bit = True # Use 4bit quantization to reduce memory usage. Can be False.

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name = "unsloth/llama-3-8b-Instruct-bnb-4bit", # Choose ANY! eg teknium/OpenHermes-2.5-Mistral-7B
    max_seq_length = max_seq_length,
    dtype = dtype,
    load_in_4bit = load_in_4bit,
    # token = "hf_...", # use one if using gated models like meta-llama/Llama-2-7b-hf
)        
Langkah 4: Menyiapkan PEFT untuk Penyetelan

Pada langkah selanjutnya, kita akan mengonfigurasi PEFT (Penyempurnaan Parameter yang Efisien) untuk mengoptimalkan proses penyempurnaan model kami. Si Model Bahasa Cepat objek di Unsloth menawarkan get_peft_model, yang memungkinkan kita untuk menyempurnakan model dengan beberapa parameter yang dapat disesuaikan.

model = FastLanguageModel.get_peft_model(
    model,
    r = 16, # Choose any number > 0 ! Suggested 8, 16, 32, 64, 128
    target_modules = ["q_proj", "k_proj", "v_proj", "o_proj",
                      "gate_proj", "up_proj", "down_proj",],
    lora_alpha = 16,
    lora_dropout = 0, # Supports any, but = 0 is optimized
    bias = "none",    # Supports any, but = "none" is optimized
    # [NEW] "unsloth" uses 30% less VRAM, fits 2x larger batch sizes!
    use_gradient_checkpointing = "unsloth", # True or "unsloth" for very long context
    random_state = 3407,
    use_rslora = False,  # We support rank stabilized LoRA
    loftq_config = None, # And LoftQ
)        
Parameter PEFT Utama untuk Mengonfigurasi:


Jumlah Kepala Perhatian: Kontrol berapa banyak kepala perhatian yang digunakan model dalam lapisan transformatornya.
Modul Target: Tentukan bagian model mana yang harus disesuaikan, memungkinkan penyesuaian yang lebih ditargetkan.
Tingkat putus sekolah: Sesuaikan probabilitas mengatur unit ke nol selama pelatihan untuk mencegah pemasangan yang berlebihan.
LoRa Alfa: Menyetel faktor penskalaan untuk adaptor peringkat rendah, yang dapat membantu model mempelajari representasi yang lebih baik.
Dan banyak lagi...
Selain itu, kami akan mengaktifkan pos pemeriksaan gradien, teknik yang membantu mengurangi penggunaan memori selama pelatihan dengan hanya menyimpan subset dari aktivasi perantara model. Hal ini memungkinkan kami untuk melatih model yang lebih besar pada perangkat keras terbatas, lebih lanjut menunjukkan kemampuan Unsloth dalam mengoptimalkan kinerja model.

Dengan PEFT yang dikonfigurasi, kami akan dapat menyempurnakan Llama3 untuk kasus penggunaan spesifik kami sambil mempertahankan kinerja dan efisiensi memori.

Langkah 5: Persiapan dan Pemformatan Data

Sebelum menyempurnakan model kita, kita perlu memberikannya akses ke kumpulan data yang relevan. Dalam hal ini, kita akan menggunakan heliosbrahma/mental_Kesehatan_Chatbot_dataset , kumpulan data pembuatan teks yang dirancang untuk membangun agen percakapan.

Standarisasi Format Data

Agar proses penyempurnaan menjadi efektif, kita perlu memastikan bahwa data disajikan dalam format yang dapat dipahami oleh model. Ini berarti mengonversi semua entri himpunan data menjadi struktur prompt yang konsisten dengan input dan output berlabel. Dengan demikian, kami menjamin bahwa model belajar dari pola yang seragam, membuat proses penyetelan lebih efisien.



Isi artikel
Image Source: My Notebook.
Anggap saja seperti mengajar seorang siswa dengan buku teks. Jika buku teks memiliki struktur yang jelas — pertanyaan diikuti dengan jawaban atau petunjuk diikuti dengan tanggapan — siswa dapat lebih mudah mempelajari materi. Dengan cara yang sama, format data standar memastikan bahwa model dapat mempelajari hubungan yang tepat antar input (Prompt) dan keluaran (Responses to).

Contoh Pemformatan Data:

{'conversations': [
{'from': 'human', 'value': 'What is a panic attack?'}, 
{'from': 'gpt', 'value': 'Panic attacks come on suddenly and involve intense and often overwhelming fear. They’re accompanied by very challenging physical symptoms, like a racing heartbeat, shortness of breath, or nausea. Unexpected panic attacks occur without an obvious cause. Expected panic attacks are cued by external stressors, like phobias. Panic attacks can happen to anyone, but having more than one may be a sign of panic disorder, a mental health condition characterized by sudden and repeated panic attacks.'}
]}        
Dengan memetakan semua titik data ke format terstruktur ini, kita dapat memastikan bahwa model dapat menggunakan himpunan data secara efektif selama fase penyempurnaan. Langkah ini sangat penting untuk memastikan model belajar dari himpunan data dengan cara yang selaras dengan aplikasi yang kita maksudkan.

Langkah 6: Melatih Model

Sekarang setelah kita menyiapkan himpunan data kita dan mengonfigurasi parameter yang diperlukan, saatnya untuk memulai proses pelatihan. Pada langkah ini, kita akan menggunakan SFTTrainer dari Unsloth untuk menyempurnakan Llama3 pada kumpulan data kami.

Menyiapkan Pelatih

Untuk memulai pelatihan, pertama-tama kami mengonfigurasi SFTTrainer, yang membutuhkan beberapa komponen utama:


Pola: Model bahasa yang kami sempurnakan.
Tokenizer: Tokenizer yang digunakan untuk memproses teks input.
Himpunan Data Pelatihan: Himpunan data yang kami siapkan pada langkah sebelumnya.
Panjang Urutan Maks: Panjang token maksimum untuk setiap urutan input.
Pemrosesan Himpunan Data: Jumlah proses paralel yang digunakan untuk memuat himpunan data.
Kemasan: Pengaturan yang dapat mempercepat pelatihan untuk urutan pendek dengan mengemasnya ke dalam batch yang lebih sedikit (kami mengatur ini ke False dalam kasus kami).
Kami juga mendefinisikan Argumen Pelatihan, yang mengontrol proses pelatihan. Ini termasuk ukuran batch, jumlah langkah akumulasi gradien, tingkat pembelajaran, pengaturan pengoptimal, dan banyak lagi.

Berikut cara kami mengkonfigurasi SFTTrainer:

trainer = SFTTrainer(
    model = model,
    tokenizer = tokenizer,
    train_dataset = dataset,
    dataset_text_field = "text",
    max_seq_length = max_seq_length,
    dataset_num_proc = 2,
    packing = False, # Can make training 5x faster for short sequences.
    args = TrainingArguments(
        per_device_train_batch_size = 2,
        gradient_accumulation_steps = 4,
        warmup_steps = 5,
        max_steps = 60,
        learning_rate = 2e-4,
        fp16 = not is_bfloat16_supported(),
        bf16 = is_bfloat16_supported(),
        logging_steps = 1,
        optim = "adamw_8bit",
        weight_decay = 0.01,
        lr_scheduler_type = "linear",
        seed = 3407,
        output_dir = "outputs",
    ),
)        
Parameter Pelatihan Utama:


Maks_langkah=60: Saya membatasi pelatihan hingga 60 langkah karena kendala komputasi pada GPU tingkat bebas kami.
per_alat_kereta api_Batch_ukuran = 2: Saya menetapkan ukuran batch 2, yang ideal untuk menyempurnakan pada GPU yang lebih kecil.
Gradien_Akumulasi_langkah=4: Ini membantu mensimulasikan ukuran batch yang lebih besar dengan mengumpulkan gradien selama 4 langkah sebelum memperbarui bobot.
Belajar_Tingkat = 2e-4: Saya menggunakan tingkat belajar sedang untuk pelatihan yang stabil.
FP16 dan BF16: Penggunaan pelatihan presisi campuran membantu mempercepat proses dan mengurangi penggunaan memori, tergantung pada dukungan perangkat keras.
optim="adam_8bit": Saya menggunakan pengoptimal AdamW dengan presisi 8-bit untuk pelatihan yang lebih hemat memori.
Setelah semuanya disiapkan, kita dapat mulai melatih model. Mengingat kendala pengaturan kami, kami akan menjalankan pelatihan hanya untuk 60 langkah. Ini cukup untuk tujuan demonstrasi, dan Anda dapat memperpanjang pelatihan jika Anda memiliki akses ke lebih banyak sumber daya komputasi.

Langkah 7: Melakukan Inferensi dan Menghasilkan Respons

Setelah model disesuaikan, saatnya untuk mengujinya dengan menghasilkan respons berdasarkan kueri input. Pada langkah ini, kita akan menggunakan yang disesuaikan Llama3 untuk menghasilkan jawaban untuk serangkaian petunjuk tertentu.

Menyiapkan Input

Pertama, kami menyiapkan pesan input. Dalam kasus kami, kami mensimulasikan percakapan di mana pengguna menanyakan tentang serangan panik. Kami memformat input menggunakan tokenizer dan menerapkan templat obrolan untuk memastikan bahwa input disusun dengan benar untuk pembangkitan.

messages = [
    {"from": "human", "value": "What is panic attack?"},
]
inputs = tokenizer.apply_chat_template(
    messages,
    tokenize = True,
    add_generation_prompt = True, # Must add for generation
    return_tensors = "pt",
).to("cuda")

outputs = model.generate(input_ids = inputs, max_new_tokens = 64, use_cache = True)
tokenizer.batch_decode(outputs)        
Model akan menghasilkan respons berdasarkan pesan input, dan Anda akan melihat teks yang dihasilkan dicetak sebagai balasan model untuk pertanyaan tersebut.

<|begin_of_text|><|start_header_id|>user<|end_header_id|>

What is panic attack?<|eot_id|><|start_header_id|>assistant<|end_header_id|>

A panic attack is a sudden and intense feeling of fear or anxiety that reaches a peak within minutes and includes physical symptoms such as a racing heartbeat, sweating, and trembling. It's a normal response to a stressful situation, but for people with panic disorder, it can happen at any time and for no apparent reason.<|eot_id|>
        
Dengan ini, kami telah berhasil menggunakan model yang disesuaikan untuk menghasilkan teks berdasarkan kueri pengguna! Model ini sekarang dapat digunakan untuk aplikasi interaktif seperti chatbot atau sistem dukungan pelanggan.

Model tersedia di Hugging Face Hub

Sekarang kita telah menyempurnakan Llama3 , langkah terakhir adalah membuatnya dapat diakses untuk digunakan dalam aplikasi dunia nyata. Model kami yang disesuaikan dihosting di Hub Wajah Memeluk di bawah repositori Rathsam/Llama-3.1-q4_k_m-kesehatan mental. Model ini telah disimpan di GGUF format, format yang sangat efisien yang dirancang untuk inferensi yang lebih cepat dan penggunaan memori yang lebih rendah.

Anda dapat dengan mudah menarik model dari Hugging Face Hub dan menjalankannya secara lokal menggunakan Ollama, alat populer untuk menjalankan model bahasa besar di mesin Anda.

Model Wajah Pelukan

Ini menyimpulkan panduan penyempurnaan kami. Anda sekarang memiliki model yang kuat dan disesuaikan dengan baik yang siap digunakan dalam berbagai aplikasi seperti chatbot kesehatan mental, agen dukungan pelanggan, atau tugas AI percakapan lainnya!

Kode Penyetelan Halus

Kesimpulan
Kesimpulannya, menyempurnakan model bahasa besar (LLM) dengan teknik seperti LoRA, QLoRA, dan Adaptor merupakan lompatan maju dalam membuat penyesuaian model dapat diakses, efisien, dan ramah sumber daya. Dengan bantuan Unsloth, kami telah melihat bagaimana metode ini memungkinkan adaptasi model yang telah dilatih sebelumnya ke tugas-tugas tertentu tanpa memerlukan sumber daya komputasi yang luas. Dengan menggunakan Penyetelan Halus yang Diawasi (SFT) dan Penyetelan Halus yang Efisien Parameter (PEFT), kami mencapai keseimbangan antara kinerja dan penggunaan sumber daya, berhasil menyempurnakan model GGUF yang sekarang dihosting di Hugging Face untuk penerapan yang dapat diskalakan.

Setiap teknik — LoRA, QLoRA, dan Adaptor — melayani tujuan yang unik, melayani kendala sumber daya dan persyaratan kinerja yang berbeda. Pembaruan peringkat rendah LoRA memberikan penyempurnaan yang efisien dengan parameter tambahan minimal, sementara QLoRA menggabungkan kuantisasi dan adaptasi peringkat rendah untuk lingkungan dengan batasan memori yang lebih ketat. Adaptor, di sisi lain, menghadirkan modularitas, membuatnya mudah untuk mengadaptasi model ke berbagai tugas sambil mempertahankan pengetahuan inti model dasar. Bersama-sama, metode ini membuka pintu baru untuk aplikasi industri di mana penyempurnaan khusus sangat penting untuk menangani tugas khusus dan dinamis.

Hasil kerja kami dengan Unsloth menggarisbawahi pentingnya teknik efisien parameter untuk penerapan model di masa depan. Karena industri semakin mengandalkan AI untuk aplikasi khusus domain, permintaan untuk penyempurnaan model yang dapat diskalakan dan hemat sumber daya hanya akan tumbuh. Teknik seperti LoRA, QLoRA, dan Adaptor tidak hanya memungkinkan untuk menerapkan LLM di lingkungan sumber daya terbatas, tetapi juga memberdayakan pengembang untuk menghadirkan model yang lebih cerdas dan responsif ke skenario dunia nyata. Perjalanan menyempurnakan, menghosting, dan berbagi model GGUF kami di Hugging Face hanyalah salah satu contoh bagaimana alat ini memungkinkan solusi bertenaga AI generasi berikutnya.

Referensi:
1. Memahami dan Menggunakan Penyempurnaan yang Diawasi (SFT) untuk Model Bahasa: https://cameronrwolfe.substack.com/p/understanding-and-using-supervised

2. Optimasi Preferensi Langsung (DPO): Mitra ringan untuk RLHF: https://toloka.ai/blog/direct-preference-optimization/

3. Apa itu adaptor?: https://www.moveworks.com/us/en/resources/ai-terms-glossary/adapters

4. Serrano.Academy: https://www.youtube.com/@SerranoAcademy/video

5. Krish Naik : Video.

6. Suman Das: Blog.

7. Datacamp: Blog.

8. Kumpulan data: heliosbrahma/mental_Kesehatan_Chatbot_Dataset.

9. ChatGPT: Untuk menyusun dan meninjau konten, saya menulis dengan pikiran yang mengalir bebas.
