<script setup>
    import { ref, onMounted } from 'vue'

    const changeTitle = ref(false)
    const changeDescription = ref(false)
    const confirmationPage = ref(false)
    const quizPage = ref(false)
    const resultPage = ref(false)

    const showBreadcrumb = ref(false)
    const breadcrumbMsg = ref('')

    const currentQuestion = ref(0)
    const currentAnswer = ref(-1)

    const action = ref('')
    
    const qna = [
        {
            "q": "Kurang tertarik atau bergairah dalam melakukan apapun.",
            "a": -1,
            "isAnswered": false
        },
        {
            "q": "Merasa murung, muram, atau putus asa.",
            "a": -1,
            "isAnswered": false
        },
        {
            "q": "Sulit tidur atau mudah terbangun, atau terlalu banyak tidur.",
            "a": -1,
            "isAnswered": false
        },
        {
            "q": "Merasa lelah atau kurang bertenaga.",
            "a": -1,
            "isAnswered": false
        },
        {
            "q": "Kurang nafsu makan atau terlalu banyak makan.",
            "a": -1,
            "isAnswered": false
        },
        {
            "q": "Kurang percaya diri — atau merasa bahwa Anda adalah orang yang gagal atau telah mengecewakan diri sendiri atau keluarga",
            "a": -1,
            "isAnswered": false
        },
        {
            "q": "Sulit berkonsentrasi pada sesuatu, misalnya membaca koran atau menonton televisi",
            "a": -1,
            "isAnswered": false
        },
        {
            "q": "Bergerak atau berbicara sangat lambat sehingga orang lain memperhatikannya. Atau sebaliknya — merasa resah atau gelisah sehingga Anda lebih sering bergerak dari biasanya.",
            "a": -1,
            "isAnswered": false
        },
        {
            "q": "Merasa lebih baik mati atau ingin melukai diri sendiri dengan cara apapun.",
            "a": -1,
            "isAnswered": false
        },
    ]

    function calculateFuzzyScore(){
        let score = 0
        qna.forEach((val) => {
            score += val.a
        })

        const rendah = Math.max(0, Math.min(1, (9 - score) / 9))
        const sedang = Math.max(0, 1 - Math.abs(score - 9.5) / 4.5)
        const tinggi = Math.max(0, Math.min(1, (score - 10) / 17))

        return {
            rendah: rendah,
            sedang: sedang,
            tinggi: tinggi
        }
    }

    function calculateFuzzyQ9(){
        const q9 = qna[qna.length - 1].a

        const tidak_risiko = q9 == 0 ? 1 : 0
        const ada_risiko = (q9 == 1 || q9 == 2) ? 1 : 0
        const tinggi_risiko = q9 == 3 ? 1 : 0

        return {
            tidak_risiko: tidak_risiko,
            ada_risiko: ada_risiko,
            tinggi_risiko: tinggi_risiko
        }
    }

    function setValue(val){
        qna[currentQuestion.value].a = val
        qna[currentQuestion.value].isAnswered = true
    }

    function nextQuestion(){
        if(currentQuestion.value < qna.length){
            currentQuestion.value++
            currentAnswer.value = qna[currentQuestion.value].a
        }
    }
    
    function prevQuestion(){
        if(currentQuestion.value > 0){
            currentQuestion.value--
            currentAnswer.value = qna[currentQuestion.value].a
        }
    }

    function checkIfCompleted(){
        let isCompleted = true

        qna.forEach((val) => {
            if(val.isAnswered == false){
                isCompleted = false
            }
        })

        return isCompleted
    }

    function breadcrumb(msg){
        breadcrumbMsg.value = msg
        showBreadcrumb.value = true

        setTimeout(() => {
            showBreadcrumb.value = false
        }, 3000)
    }

    function finishQuiz(){
        let check = checkIfCompleted()

        if(!check){
            breadcrumb('⚠️ Masih ada soal yang tertinggal! Harap cek kembali jawaban anda.')
        }
        else{
            let fuzzyScore = calculateFuzzyScore()
            let fuzzyQ9 = calculateFuzzyQ9()

            let result = [
                {
                    action: 'Lanjut',
                    weight: Math.min(fuzzyScore.rendah, fuzzyQ9.tidak_risiko),
                },
                {
                    action: 'Rujuk',
                    weight: Math.max(
                        Math.min(fuzzyScore.sedang, fuzzyQ9.tidak_risiko),
                        Math.min(fuzzyScore.tinggi, fuzzyQ9.tidak_risiko),
                        Math.min(fuzzyScore.rendah, fuzzyQ9.ada_risiko),
                        Math.min(fuzzyScore.sedang, fuzzyQ9.ada_risiko),
                    ),
                },
                {
                    action: 'Rujuk Segera',
                    weight: Math.max(
                        fuzzyQ9.tinggi_risiko,
                        Math.min(fuzzyScore.tinggi, fuzzyQ9.ada_risiko)
                    )
                }
            ]

            action.value = result.sort(function(a, b){ return b.weight - a.weight })[0].action
            resultPage.value = true
        }
    }

    onMounted(async() => {
        await setTimeout(() => {
            changeTitle.value = true
        }, 2000)
    })  
</script>

<template>
    <div class="w-full h-screen flex items-center justify-center bg-[#F0E4D3] relative">
        <Transition name="slide-down" mode="out-in">
            <div class="bg-[#493628] p-4 rounded-xl text-[#fff9f2] w-1/2 flex gap-1 z-20 absolute top-5 left-1/4" v-if="showBreadcrumb">
                <p>{{ breadcrumbMsg }}</p>
            </div>
        </Transition>

        <Transition name="fade" mode="out-in" v-if="!resultPage">
            <Transition name="fade" mode="out-in" v-if="!quizPage">
                <Transition name="fade" mode="out-in" v-if="!confirmationPage">
                    <Transition name="fade" mode="out-in" v-if="!changeDescription">
                        <div class="w-full" v-if="!changeTitle">
                            <p class="text-6xl font-semibold text-[#493628] text-center">Halo.</p>
                        </div>

                        <div class="w-full flex flex-col gap-6 items-center justify-center" v-else>
                            <p class="text-6xl font-semibold text-[#AB886D] text-center">Selamat datang di <span class="text-[#493628]">Helping Hand</span>.</p>
                            <p class="text-center w-1/2 text-lg text-[#493628]">Di Helping Hand, kami percaya bahwa mengambil langkah pertama untuk memahami kesejahteraan mental Anda harus terasa sederhana, mendukung, dan bebas stigma.</p>

                            <button class="px-4 py-2 rounded-lg bg-[#AB886D] hover:bg-[#493628] hover:-translate-y-1 text-white transition-all duration-300 ease-in-out text-lg" @click="() => changeDescription = true">Lanjut</button>
                        </div>
                    </Transition>
                    
                    <div class="w-full flex flex-col gap-6 items-center justify-center" v-else>
                        <p class="text-6xl font-semibold text-[#AB886D] text-center"><span class="text-[#493628]">Penilaian</span> apa ini?</p>
                        <p class="text-center w-2/3 text-lg text-[#493628]">Anda akan memulai penilaian diri singkat berdasarkan <span class="font-semibold">PHQ-9</span> (Patient Health Questionnaire-9), sebuah alat tepercaya yang digunakan oleh para profesional di seluruh dunia untuk mengenali tanda-tanda depresi. Pemeriksaan ini mencakup <span class="font-semibold">9 pertanyaan</span> tentang perasaan Anda selama <span class="font-semibold">dua minggu terakhir</span>. Hanya perlu beberapa menit, dan jawaban Anda akan membantu memandu perawatan dan percakapan yang bermakna.</p>

                        <button class="px-4 py-2 rounded-lg bg-[#AB886D] hover:bg-[#493628] hover:-translate-y-1 text-white transition-all duration-300 ease-in-out text-lg" @click="() => confirmationPage = true">Lanjut</button>
                    </div>
                </Transition>

                <div class="w-full flex flex-col gap-6 items-center justify-center" v-else>
                    <p class="text-6xl font-semibold text-[#AB886D] text-center"><span class="text-[#493628]">Sebelum</span> anda memulai...</p>    
                    <p class="text-center w-2/3 text-lg text-[#493628]">Luangkan waktu sejenak untuk membaca catatan singkat ini sebelum memulai penilaian.</p>

                    <div class="bg-[#fff9f2] p-8 rounded-xl text-[#493628] w-1/2">
                        <ul>
                            <li>📅 Renungkan apa yang Anda rasakan selama 2 minggu terakhir.</li>
                            <li>🧘 Jawablah di tempat yang tenang dan sepi jika memungkinkan.</li>
                            <li>💬 Tidak ada jawaban yang benar atau salah - jujurlah dengan diri Anda sendiri.</li>
                            <li>🔒 Tanggapan Anda bersifat pribadi dan tidak akan dibagikan tanpa seizin Anda.</li>
                            <li>⏲️ Hanya membutuhkan waktu sekitar 2 menit untuk menyelesaikannya.</li>
                        </ul>
                    </div>

                    <div class="bg-[#493628] p-4 rounded-xl text-[#fff9f2] w-1/2 flex gap-1">
                        <span>⚠️</span>
                        <p><span class="font-semibold">Harap diperhatikan</span> bahwa hasil ini bukan merupakan diagnosis, tetapi merupakan evaluasi awal untuk mendukung diskusi lebih lanjut dengan seorang profesional berlisensi. Jika Anda merasa tertekan atau berada dalam krisis, kami menganjurkan Anda untuk <span class="font-semibold">segera mencari bantuan dari penyedia layanan kesehatan mental atau layanan dukungan darurat di daerah Anda.</span></p>
                    </div>

                    <button class="px-4 py-2 rounded-lg bg-[#AB886D] hover:bg-[#493628] hover:-translate-y-1 text-white transition-all duration-300 ease-in-out text-lg" @click="() => quizPage = true">Mulai</button>
                </div>
            </Transition>

            <div class="w-full flex flex-col gap-20 items-center justify-center" v-else>
                <div class="w-full flex items-center justify-center gap-3">
                    <div class="w-[2em] h-[0.5em] transition-all duration-300 ease-in-out rounded-lg" :class="index == currentQuestion ? 'bg-[#AB886D]' : x.isAnswered ? 'bg-[#493628]' : 'bg-[#fff9f2]'" v-for="(x, index) in qna"></div>
                </div>

                <div class="w-2/3 flex flex-col gap-10">
                    <p class="text-5xl leading-12 font-semibold text-[#493628]">Selama 2 minggu terakhir, seberapa sering Anda terganggu oleh masalah berikut?</p>

                    <Transition name="fade" mode="out-in">
                        <div class="w-full" :key="qna[currentQuestion].q">
                            <p class="text-2xl font-medium text-[#493628] pb-5">{{ qna[currentQuestion].q }}</p>

                            <div class="flex gap-2 pb-2">
                                <input type="radio" name="answer" id="answer0" v-model.number="currentAnswer" value="0" @click="() => setValue(0)"><label for="answer0" class="text-xl">Tidak pernah</label>
                            </div>
                            <div class="flex gap-2 pb-2">
                                <input type="radio" name="answer" id="answer1" v-model.number="currentAnswer" value="1" @click="() => setValue(1)"><label for="answer1" class="text-xl">Beberapa hari</label>
                            </div>
                            <div class="flex gap-2 pb-2">
                                <input type="radio" name="answer" id="answer2" v-model.number="currentAnswer" value="2" @click="() => setValue(2)"><label for="answer2" class="text-xl">Lebih dari separuh waktu yang dimaksud</label>
                            </div>
                            <div class="flex gap-2 pb-2">
                                <input type="radio" name="answer" id="answer3" v-model.number="currentAnswer" value="3" @click="() => setValue(3)"><label for="answer3" class="text-xl">Hampir setiap hari</label>
                            </div>
                        </div>
                    </Transition>
                </div>

                <div class="w-2/3 flex items-center justify-between">
                    <button class="px-4 py-2 rounded-lg bg-[#AB886D] hover:bg-[#493628] hover:-translate-y-1 text-white transition-all duration-300 ease-in-out text-lg" @click="() => prevQuestion()">Soal sebelumnya</button>

                    <button class="px-4 py-2 rounded-lg bg-[#AB886D] hover:bg-[#493628] hover:-translate-y-1 text-white transition-all duration-300 ease-in-out text-lg" @click="() => nextQuestion()" v-if="currentQuestion < qna.length - 1">Soal selanjutnya</button>

                    <button class="px-4 py-2 rounded-lg bg-[#AB886D] hover:bg-[#493628] hover:-translate-y-1 text-white transition-all duration-300 ease-in-out text-lg" @click="() => finishQuiz()" v-if="currentQuestion == qna.length - 1">Selesaikan</button>
                </div>
            </div>
        </Transition>

        <div class="w-2/3 flex flex-col gap-3 justify-center" v-else>
            <p class="text-[#493628] font-medium text-2xl pb-2">Hasil:</p>
            <p class="text-5xl leading-12 font-semibold text-[#493628] pb-10">
                {{ 
                    action == 'Lanjut' ? '🟢 Kategori: Rendah — Lanjutkan Pengawasan' :
                    action == 'Rujuk' ? '🟡 Kategori: Sedang — Rujuk' :
                    '🔴 Kategori: Tinggi — Rujuk Segera'
                }}
            </p>
            <div class="text-xl text-[#493628] flex flex-col gap-2" v-if="action == 'Lanjut'">
                <p>Hasil evaluasi menunjukkan tingkat depresi yang <span class="font-semibold">rendah</span>.</p>
                <p>Berdasarkan skor PHQ-9 Anda, saat ini tidak terdapat indikasi gangguan depresi yang signifikan. Meskipun demikian, penting untuk tetap melakukan pemantauan berkala terhadap kondisi psikologis Anda, terutama jika muncul gejala baru atau perubahan signifikan dalam perasaan dan fungsi harian.</p>
                <p><span class="font-semibold">Rekomendasi</span>: Tidak diperlukan rujukan saat ini. Namun, tetap jaga pola hidup sehat dan keseimbangan emosional. Konsultasikan dengan tenaga profesional apabila gejala mulai berkembang.</p>
            </div>
            <div class="text-xl text-[#493628] flex flex-col gap-2" v-if="action == 'Rujuk'">
                <p>Hasil evaluasi menunjukkan indikasi gejala depresi tingkat <span class="font-semibold">sedang</span>.</p>
                <p>Skor PHQ-9 Anda mengindikasikan adanya gangguan yang mungkin memengaruhi kesejahteraan psikologis dan fungsi harian. Kondisi ini berpotensi berkembang jika tidak ditangani secara tepat.</p>
                <p><span class="font-semibold">Rekomendasi</span>: Disarankan untuk segera menjadwalkan konsultasi dengan tenaga kesehatan jiwa (psikolog atau psikiater) guna mendapatkan asesmen klinis lanjutan dan pertimbangan intervensi yang sesuai.</p>
            </div>
            <div class="text-xl text-[#493628] flex flex-col gap-2" v-if="action == 'Rujuk Segera'">
                <p>Hasil evaluasi menunjukkan kemungkinan besar adanya gangguan depresi <span class="font-semibold">berat</span>.</p>
                <p>Skor PHQ-9 Anda berada pada tingkat yang mengkhawatirkan, dengan kemungkinan besar terdapat dampak signifikan terhadap keseharian dan fungsi psikososial. <span class="font-semibold">Kondisi ini memerlukan perhatian segera</span>.</p>
                <p><span class="font-semibold">Rekomendasi</span>: Segera hubungi profesional kesehatan jiwa untuk asesmen mendalam dan penanganan klinis yang tepat. Penundaan dalam mendapatkan bantuan dapat memperburuk kondisi. Apabila terdapat pikiran menyakiti diri sendiri atau bunuh diri, <span class="font-semibold">segera cari bantuan darurat</span>.</p>
            </div>

            <button class="px-4 py-2 rounded-lg bg-[#AB886D] hover:bg-[#493628] hover:-translate-y-1 text-white transition-all duration-300 ease-in-out text-lg w-fit" @click="() => resultPage = false">Kembali</button>
        </div>
    </div>
</template>

<style>
    .fade-enter-active,
    .fade-leave-active {
        transition: opacity 0.5s ease;
    }

    .fade-enter-from,
    .fade-leave-to {
        opacity: 0;
    }

    .slide-down-enter-active,
    .slide-down-leave-active {
        transition: all 300ms ease;
    }

    .slide-down-enter-from {
        opacity: 0;
        transform: translateY(-10px);
    }

    .slide-down-leave-to {
        opacity: 0;
        transform: translateY(10px);
    }
</style>