// bookmarklet.js - Script para criar uma caixa flutuante com um iframe de chat

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
        widget.className = 'fixed top-1/4 left-1/2 -translate-x-1/2 bg-gray-800 text-white rounded-lg shadow-2xl z-[9999] w-80 md:w-96 overflow-hidden flex flex-col resize-x resize-y'; // Added resize-x and resize-y for resizing in both directions
        widget.style.minWidth = '300px';
        widget.style.minHeight = '300px';
        widget.style.maxHeight = '90vh'; // Added maxHeight to limit vertical expansion
        widget.style.maxWidth = '90vw'; // Added maxWidth for horizontal expansion
        widget.style.cursor = 'grab'; // Indicate it's draggable
        widget.style.display = 'flex'; // Ensure flexbox for layout
        widget.style.flexDirection = 'column'; // Stack elements vertically
        widget.style.setProperty('--tw-shadow', '0 25px 50px -12px rgba(0, 0, 0, 0.25), 0 0 0 1px rgba(0, 0, 0, 0.05)');


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

        // 3. Cria o corpo do widget (iframe de chat)
        const body = document.createElement('div');
        body.id = 'ai-chat-body';
        body.className = 'flex-grow p-0 flex flex-col overflow-hidden'; // flex-grow para ocupar o espaço restante
        body.innerHTML = `
            <iframe id="chat-iframe"
                    src="" <!-- COLOQUE O LINK DO SEU SITE DE CHAT GPT AQUI -->
                    class="w-full h-full border-0 rounded-b-lg"></iframe>
        `;
        widget.appendChild(body);

        document.body.appendChild(widget);

        // Referências aos elementos
        const minimizeBtn = document.getElementById('minimize-btn');
        const closeBtn = document.getElementById('close-btn');
        const chatIframe = document.getElementById('chat-iframe'); // Referência ao iframe

        // ATENÇÃO: COLOQUE O LINK DO SEU SITE DE CHAT GPT AQUI
        // Exemplo: chatIframe.src = "https://chat.openai.com/";
        // Verifique se o site permite ser embedado em um iframe (alguns sites podem bloquear por segurança).
        chatIframe.src = "https://gemini.google.com/"; // Exemplo padrão, mude para o que desejar.

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
            widget.style.right = 'auto'; // Desabilita o posicionamento automático para direita/inferior
            widget.style.bottom = 'auto'; // Desabilita o posicionamento automático para direita/inferior
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
                widget.style.height = `${header.offsetHeight}px`; // Colapsa para a altura do cabeçalho
                widget.style.maxHeight = 'none'; // Permite colapsar abaixo da altura máxima
                widget.style.width = '280px'; // Define uma largura fixa quando minimizado
                widget.style.minWidth = '280px';
                widget.classList.remove('resize-x', 'resize-y'); // Remove resize quando minimizado
            } else {
                body.style.display = 'flex';
                widget.style.height = ''; // Restaura a altura automática
                widget.style.maxHeight = '80vh'; // Restaura a altura máxima
                widget.style.width = ''; // Restaura a largura automática
                widget.style.minWidth = '300px';
                widget.classList.add('resize-x', 'resize-y'); // Adiciona o resize de volta
            }
        });

        // Função para fechar o widget
        closeBtn.addEventListener('click', () => {
            widget.remove();
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
        widget.style.top = `${(window.innerHeight - widget.offsetHeight) / 2}px`;
        widget.style.left = `${(window.innerWidth - widget.offsetWidth) / 2}px`;

        // Observa mudanças de tamanho no widget (para redimensionamento manual pelo usuário)
        const resizeObserver = new ResizeObserver(entries => {
            for (let entry of entries) {
                if (entry.target === widget) {
                    // console.log(`Widget resized to ${entry.contentRect.width}x${entry.contentRect.height}`);
                    // Garante que o corpo do widget e o iframe também se redimensionem
                    body.style.width = `${entry.contentRect.width}px`;
                    body.style.height = `${entry.contentRect.height - header.offsetHeight}px`;
                    chatIframe.style.width = '100%';
                    chatIframe.style.height = '100%';
                }
            }
        });
        resizeObserver.observe(widget);

        console.log("Widget de chat AI com iframe criado e inicializado.");
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

