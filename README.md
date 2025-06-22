// bookmarklet.js - Script para criar uma caixa flutuante de chat AI

(function() {
    // Carrega Tailwind CSS dinamicamente
    if (!document.getElementById('tailwind-script')) {
        const tailwindScript = document.createElement('script');
        tailwindScript.src = "https://cdn.tailwindcss.com";
        tailwindScript.id = "tailwind-script";
        document.head.appendChild(tailwindScript);
    }

    // Espera o Tailwind CSS carregar antes de criar o widget para garantir que as classes funcionem
    const initializeWidget = () => {
        // Verifica se o widget já existe para evitar duplicações
        if (document.getElementById('ai-chat-widget')) {
            console.log("Widget de chat AI já existe na página.");
            return;
        }

        // 1. Cria o elemento principal do widget
        const widget = document.createElement('div');
        widget.id = 'ai-chat-widget';
        widget.className = 'fixed top-1/4 left-1/2 -translate-x-1/2 bg-gray-800 text-white rounded-lg shadow-2xl z-[9999] w-80 md:w-96 overflow-hidden flex flex-col resize-x'; // Added resize-x for horizontal resizing
        widget.style.minWidth = '280px';
        widget.style.minHeight = '200px'; // Changed to minHeight
        widget.style.maxHeight = '80vh'; // Added maxHeight to limit vertical expansion
        widget.style.maxWidth = '90vw'; // Added maxWidth for horizontal expansion
        widget.style.cursor = 'grab'; // Indicate it's draggable
        widget.style.display = 'flex'; // Ensure flexbox for layout
        widget.style.flexDirection = 'column'; // Stack elements vertically

        // 2. Cria o cabeçalho do widget
        const header = document.createElement('div');
        header.className = 'flex justify-between items-center bg-gray-700 p-2 rounded-t-lg cursor-grab select-none';
        header.innerHTML = `
            <h2 class="text-lg font-bold">Chat AI</h2>
            <div class="flex space-x-2">
                <button id="minimize-btn" class="p-1 rounded-full hover:bg-gray-600 focus:outline-none">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 12H4"></path></svg>
                </button>
                <button id="close-btn" class="p-1 rounded-full hover:bg-red-500 focus:outline-none">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                </button>
            </div>
        `;
        widget.appendChild(header);

        // 3. Cria o corpo do widget (conteúdo do chat)
        const body = document.createElement('div');
        body.id = 'ai-chat-body';
        body.className = 'flex-grow p-4 flex flex-col space-y-4 overflow-y-auto'; // Use flex-grow and overflow-y-auto
        body.innerHTML = `
            <div class="flex-grow flex flex-col space-y-2">
                <label for="question-input" class="text-sm font-semibold">Sua Questão:</label>
                <textarea id="question-input"
                          class="w-full p-2 rounded-md bg-gray-700 border border-gray-600 focus:ring-blue-500 focus:border-blue-500 text-sm resize-y"
                          rows="4"
                          placeholder="Digite sua pergunta aqui..."></textarea>
                <button id="send-btn"
                        class="mt-2 bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-md shadow-lg transition duration-300 ease-in-out transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-75">
                    Enviar
                </button>
            </div>
            <div class="flex-grow flex flex-col space-y-2 mt-4 border-t border-gray-700 pt-4">
                <label for="answer-output" class="text-sm font-semibold">Resposta da IA:</label>
                <div id="answer-output"
                     class="w-full p-2 rounded-md bg-gray-700 border border-gray-600 text-sm overflow-y-auto h-32 md:h-48">
                    <!-- Respostas da IA aparecerão aqui -->
                </div>
                <div id="loading-indicator" class="text-blue-400 text-sm hidden">Gerando resposta...</div>
            </div>
        `;
        widget.appendChild(body);

        document.body.appendChild(widget);

        // Referências aos elementos
        const minimizeBtn = document.getElementById('minimize-btn');
        const closeBtn = document.getElementById('close-btn');
        const questionInput = document.getElementById('question-input');
        const sendBtn = document.getElementById('send-btn');
        const answerOutput = document.getElementById('answer-output');
        const loadingIndicator = document.getElementById('loading-indicator');

        let isDragging = false;
        let offset = { x: 0, y: 0 };
        let isMinimized = false;

        // Função para arrastar o widget
        header.addEventListener('mousedown', (e) => {
            isDragging = true;
            offset = {
                x: e.clientX - widget.offsetLeft,
                y: e.clientY - widget.offsetTop
            };
            widget.style.cursor = 'grabbing';
        });

        document.addEventListener('mousemove', (e) => {
            if (!isDragging) return;
            widget.style.left = (e.clientX - offset.x) + 'px';
            widget.style.top = (e.clientY - offset.y) + 'px';
            widget.style.right = 'auto'; // Disable auto positioning for right/bottom
            widget.style.bottom = 'auto'; // Disable auto positioning for right/bottom
        });

        document.addEventListener('mouseup', () => {
            isDragging = false;
            widget.style.cursor = 'grab';
        });

        // Função para minimizar/restaurar o widget
        minimizeBtn.addEventListener('click', () => {
            isMinimized = !isMinimized;
            if (isMinimized) {
                body.style.display = 'none';
                widget.style.height = `${header.offsetHeight}px`; // Collapse to header height
                widget.style.maxHeight = 'none'; // Allow collapsing below max height
                widget.style.width = '280px'; // Set a fixed width when minimized
                widget.style.minWidth = '280px';
                widget.classList.remove('resize-x'); // Remove resize when minimized
            } else {
                body.style.display = 'flex';
                widget.style.height = ''; // Restore auto height
                widget.style.maxHeight = '80vh'; // Restore max height
                widget.style.width = ''; // Restore auto width
                widget.style.minWidth = '280px';
                widget.classList.add('resize-x'); // Add resize back
            }
        });

        // Função para fechar o widget
        closeBtn.addEventListener('click', () => {
            widget.remove();
        });

        // Função para enviar a pergunta para o modelo AI
        sendBtn.addEventListener('click', async () => {
            const prompt = questionInput.value.trim();
            if (!prompt) {
                answerOutput.innerHTML = '<p class="text-red-400">Por favor, digite sua pergunta.</p>';
                return;
            }

            answerOutput.innerHTML = ''; // Limpa a resposta anterior
            loadingIndicator.classList.remove('hidden'); // Mostra o indicador de carregamento
            sendBtn.disabled = true; // Desabilita o botão enquanto carrega

            try {
                let chatHistory = [];
                chatHistory.push({ role: "user", parts: [{ text: prompt }] });
                const payload = { contents: chatHistory };
                // ATENÇÃO: Para usar modelos diferentes de gemini-2.0-flash,
                // você precisará de uma chave de API válida.
                // Para produção, considere usar um proxy para proteger sua chave de API.
                const apiKey = ""; // <--- Sua chave de API aqui (ou deixe vazio para o ambiente Canvas)
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                const result = await response.json();

                if (result.candidates && result.candidates.length > 0 &&
                    result.candidates[0].content && result.candidates[0].content.parts &&
                    result.candidates[0].content.parts.length > 0) {
                    const text = result.candidates[0].content.parts[0].text;
                    answerOutput.innerHTML = `<p>${text.replace(/\n/g, '<br>')}</p>`; // Exibe a resposta
                } else {
                    console.error("Estrutura de resposta inesperada ou conteúdo ausente:", result);
                    answerOutput.innerHTML = '<p class="text-red-400">Erro ao obter resposta da IA. Verifique a chave da API ou a resposta do servidor.</p>';
                }
            } catch (error) {
                console.error("Erro ao chamar a API Gemini:", error);
                answerOutput.innerHTML = `<p class="text-red-400">Ocorreu um erro: ${error.message}.</p>`;
            } finally {
                loadingIndicator.classList.add('hidden'); // Esconde o indicador de carregamento
                sendBtn.disabled = false; // Habilita o botão novamente
            }
        });

        // Posiciona o widget no centro da tela e ajusta para o viewport se necessário
        const adjustWidgetPosition = () => {
            const widgetRect = widget.getBoundingClientRect();
            if (widgetRect.right > window.innerWidth) {
                widget.style.left = (window.innerWidth - widgetRect.width) / 2 + 'px';
            }
            if (widgetRect.bottom > window.innerHeight) {
                widget.style.top = (window.innerHeight - widgetRect.height) / 2 + 'px';
            }
        };

        // Ajusta a posição ao carregar e ao redimensionar a janela
        adjustWidgetPosition();
        window.addEventListener('resize', adjustWidgetPosition);

        // Centraliza o widget na tela (opcional, pois o CSS já faz um pouco disso)
        // Isso recalcula a posição no caso de o viewport mudar de tamanho
        widget.style.top = `${(window.innerHeight - widget.offsetHeight) / 2}px`;
        widget.style.left = `${(window.innerWidth - widget.offsetWidth) / 2}px`;

        // Observa mudanças de tamanho no widget (para redimensionamento manual pelo usuário)
        const resizeObserver = new ResizeObserver(entries => {
            for (let entry of entries) {
                if (entry.target === widget) {
                    // console.log(`Widget resized to ${entry.contentRect.width}x${entry.contentRect.height}`);
                    // Ensure body also resizes with the widget
                    body.style.height = `${entry.contentRect.height - header.offsetHeight}px`;
                }
            }
        });
        resizeObserver.observe(widget);

        console.log("Widget de chat AI criado e inicializado.");
    };

    // Verifica se Tailwind já carregou, senão espera
    if (typeof tailwindcss !== 'undefined') {
        initializeWidget();
    } else {
        const tailwindCheckInterval = setInterval(() => {
            if (typeof tailwindcss !== 'undefined') {
                clearInterval(tailwindCheckInterval);
                initializeWidget();
            }
        }, 100); // Checa a cada 100ms
    }
})();
