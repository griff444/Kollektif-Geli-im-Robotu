<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kolektif Gelişim Robotu</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        #chat-container::-webkit-scrollbar { width: 8px; }
        #chat-container::-webkit-scrollbar-track { background: #f1f5f9; }
        #chat-container::-webkit-scrollbar-thumb { background: #94a3b8; border-radius: 4px; }
        .chat-bubble-user { background-color: #3b82f6; color: white; }
        .chat-bubble-bot { background-color: #e5e7eb; color: #1f2937; }
        .chat-bubble-teach { background-color: #fef9c3; border-left: 4px solid #f59e0b; }
    </style>
</head>
<body class="bg-gray-100 dark:bg-gray-900">
    <div class="flex flex-col h-screen max-w-3xl mx-auto bg-white dark:bg-gray-800 shadow-2xl">
        <header class="p-4 border-b dark:border-gray-700 flex justify-between items-center">
            <div>
                <h1 class="text-2xl font-bold text-gray-800 dark:text-white">Kolektif Gelişim Robotu</h1>
                <p class="text-sm text-gray-500 dark:text-gray-400">Beynimiz, hepimizin öğrettikleriyle dolu.</p>
            </div>
            <button id="clear-memory" class="px-3 py-2 text-xs font-medium text-white bg-red-600 rounded-lg hover:bg-red-700 transition-all">Hafızayı Temizle</button>
        </header>

        <main id="chat-container" class="flex-1 p-6 overflow-y-auto space-y-4">
            <!-- Mesajlar buraya eklenecek -->
        </main>

        <footer class="p-4 border-t dark:border-gray-700">
            <form id="chat-form" class="flex items-center space-x-2">
                <input type="text" id="message-input" placeholder="Bir mesaj yaz..." class="flex-1 p-3 border rounded-full focus:ring-2 focus:ring-blue-500 focus:outline-none dark:bg-gray-700 dark:text-white dark:border-gray-600" autocomplete="off">
                <button type="submit" class="p-3 bg-blue-500 text-white rounded-full hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" /></svg>
                </button>
            </form>
        </footer>
    </div>

    <script type="module">
        // Firebase kütüphanelerini içe aktar
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getFirestore, doc, setDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";

        const chatContainer = document.getElementById('chat-container');
        const chatForm = document.getElementById('chat-form');
        const messageInput = document.getElementById('message-input');
        const clearMemoryBtn = document.getElementById('clear-memory');

        // TEMEL ÇEKİRDEK HAFIZA (Veritabanı boşsa bununla başlar)
        const coreMemory = {
            "merhaba": "Merhaba sana da!",
            "selam": "Selam, hoş geldin!",
            "naber": "İyidir, hepimiz sayesinde gelişiyorum. Senden naber?",
            "nasılsın": "Harikayım, her gün yeni şeyler öğreniyorum. Sen nasılsın?",
            "adın ne": "Benim bir adım yok, ben hepimizin ortak zekasıyım.",
            "kimsin sen": "Ben, herkesin öğrettikleriyle gelişen kolektif bir sohbet robotuyum."
        };

        let learnedData = {}; // Lokal beyin, Firestore ile senkronize olacak
        let db; // Firestore veritabanı nesnesi
        let brainDocRef; // Beyin verisinin tutulduğu belge referansı
        let confirmClear = false; // Hafıza temizleme onayı için

        // Canvas ortamından gelen Firebase yapılandırmasını kullan
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

        async function initializeFirebase() {
            const app = initializeApp(firebaseConfig);
            db = getFirestore(app);
            const auth = getAuth(app);

             // Firebase'e giriş yap (Canvas ortamı için)
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
            } catch (error) {
                console.error("Authentication failed:", error);
                appendMessage("Sunucuya bağlanırken bir hata oluştu. Lütfen sayfayı yenileyin.", "bot");
                return;
            }

            // Herkesin erişebileceği, paylaşılan beynin veritabanı yolu
            brainDocRef = doc(db, `/artifacts/${appId}/public/data/chatbotBrain`, "sharedKnowledge");
            
            // Beyindeki değişiklikleri GERÇEK ZAMANLI olarak dinle
            onSnapshot(brainDocRef, (docSnap) => {
                if (docSnap.exists()) {
                    learnedData = docSnap.data(); // Lokal beyni güncelle
                    console.log("Beyin güncellendi!");
                } else {
                    // Eğer beyin hiç oluşturulmamışsa, çekirdek hafıza ile başlat
                    setDoc(brainDocRef, coreMemory);
                    learnedData = coreMemory;
                }
            }, (error) => {
                console.error("Failed to listen to brain changes:", error);
                appendMessage("Beyin bağlantısında bir hata oluştu. Sayfayı yenilemeyi deneyin.", "bot");
            });

            appendMessage("Merhaba! Ben kolektif gelişim robotu. Bana öğrettiğin her şeyi herkes görebilir ve kullanabilir.", "bot");
        }


        chatForm.addEventListener('submit', (e) => {
            e.preventDefault();
            const userInput = messageInput.value.trim();
            if (userInput === '' || !db) return;

            appendMessage(userInput, 'user');
            messageInput.value = '';
            getBotResponse(userInput);
        });

        clearMemoryBtn.addEventListener('click', () => {
            if (!confirmClear) {
                confirmClear = true;
                clearMemoryBtn.textContent = "Emin misin? Tıkla ve Sil";
                clearMemoryBtn.classList.replace('bg-red-600', 'bg-yellow-500');
                setTimeout(() => {
                    confirmClear = false;
                    clearMemoryBtn.textContent = "Hafızayı Temizle";
                    clearMemoryBtn.classList.replace('bg-yellow-500', 'bg-red-600');
                }, 3000); // 3 saniye içinde tıklanmazsa normale dön
            } else {
                 if (!db) return;
                 setDoc(brainDocRef, coreMemory)
                    .then(() => {
                        chatContainer.innerHTML = '';
                        appendMessage("Kolektif hafıza temizlendi. Her şeye yeniden başlıyoruz.", "bot");
                    })
                    .catch(e => console.error("Hafıza temizlenemedi:", e));
                 
                confirmClear = false;
                clearMemoryBtn.textContent = "Hafızayı Temizle";
                clearMemoryBtn.classList.replace('bg-yellow-500', 'bg-red-600');
            }
        });

        function appendMessage(text, sender, isTeachPrompt = false, originalQuestion = '') {
            const messageWrapper = document.createElement('div');
            messageWrapper.classList.add('flex', sender === 'user' ? 'justify-end' : 'justify-start');
            
            const messageDiv = document.createElement('div');
            messageDiv.classList.add('p-3', 'rounded-2xl', 'max-w-md', 'break-words');
            
            if (sender === 'user') {
                messageDiv.classList.add('chat-bubble-user');
                messageDiv.textContent = text;
            } else {
                if(isTeachPrompt){
                    messageDiv.classList.add('chat-bubble-teach');
                    messageDiv.innerHTML = `<p class="font-medium">"${originalQuestion}" demene karşılık ne demeliyim? Lütfen bana öğret.</p>`;
                    addTeachForm(messageDiv, originalQuestion);
                } else {
                    messageDiv.classList.add('chat-bubble-bot');
                    messageDiv.textContent = text;
                }
            }

            messageWrapper.appendChild(messageDiv);
            chatContainer.appendChild(messageWrapper);
            chatContainer.scrollTop = chatContainer.scrollHeight;
        }

        function addTeachForm(container, question){
            const teachForm = document.createElement('form');
            teachForm.className = 'w-full mt-2 flex space-x-2';
            teachForm.innerHTML = `
                <input type="text" placeholder="Doğru cevap..." class="flex-1 p-2 text-sm border rounded-full dark:bg-gray-600 dark:text-white dark:border-gray-500">
                <button type="submit" class="px-3 py-1 text-sm bg-green-500 text-white rounded-full hover:bg-green-600">Öğret</button>
            `;
            container.appendChild(teachForm);
            
            teachForm.querySelector('input').focus();

            teachForm.addEventListener('submit', (e) => {
                e.preventDefault();
                const newAnswer = teachForm.querySelector('input').value.trim();
                if (newAnswer) {
                    learn(question, newAnswer);
                    container.innerHTML = `<p class="text-xs text-green-700 italic">Teşekkürler, öğrendim! Bu bilgi artık herkes tarafından kullanılabilir.</p>`;
                }
            });
        }
        
        async function learn(question, answer) {
            if (!db) return;
            const key = question.toLowerCase().trim();
            try {
                // setDoc ile merge:true kullanarak sadece ilgili soruyu ekle/güncelle
                await setDoc(brainDocRef, { [key]: answer }, { merge: true });
            } catch (error) {
                console.error("Öğrenme hatası:", error);
            }
        }

        function getBotResponse(userInput) {
            const questionKey = userInput.toLowerCase().trim();
            
            setTimeout(() => {
                if (learnedData && learnedData[questionKey]) {
                    appendMessage(learnedData[questionKey], 'bot');
                } else {
                    appendMessage('', 'bot', true, userInput);
                }
            }, 500);
        }

        // Firebase'i başlat
        initializeFirebase();
    </script>
</body>
</html>
