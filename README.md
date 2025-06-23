<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel de Perguntas</title>
    <!-- Tailwind CSS CDN para facilitar o estilo -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Fonte Inter para consistência -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        /* Estilos para o sistema de notificações (do seu script original) */
        .notification-container {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 9999;
            display: flex;
            flex-direction: column;
            align-items: flex-end;
            pointer-events: none;
        }
        .notification {
            background: rgba(20, 20, 20, 0.9);
            color: #f0f0f0;
            margin-bottom: 10px;
            padding: 12px 18px;
            border-radius: 8px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.3);
            backdrop-filter: blur(8px);
            font-family: 'Inter', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
            font-size: 13.5px;
            width: 280px;
            min-height: 50px;
            text-align: center;
            display: flex;
            align-items: center;
            position: relative;
            overflow: hidden;
            pointer-events: auto;
            opacity: 0;
            transform: translateY(-20px);
            transition: opacity 0.3s ease, transform 0.3s ease;
        }
        .notification.show {
            opacity: 1;
            transform: translateY(0);
        }
        .notification-icon {
            margin-right: 10px;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .notification-progress {
            position: absolute;
            bottom: 0;
            left: 0;
            height: 3px;
            width: 100%;
            background: #f0f0f0;
            opacity: 0.8;
        }
        @keyframes progress-animation {
            from { width: 100%; }
            to { width: 0%; }
        }
        .notification-progress.animate {
            animation: progress-animation linear forwards;
        }
        .notification.success .notification-icon {
            color: #4caf50;
        }
        .notification.error .notification-icon {
            color: #f44336;
        }
        .notification.info .notification-icon {
            color: #2196f3;
        }
        .notification.warning .notification-icon {
            color: #ff9800;
        }

        /* Estilos para o novo painel */
        .question-panel {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%); /* Centraliza o painel */
            width: 380px;
            background-color: rgba(30, 30, 40, 0.95); /* Cor de fundo mais escura */
            color: #f0f0f0;
            border-radius: 12px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.4);
            backdrop-filter: blur(10px);
            z-index: 10000; /* Acima das notificações */
            resize: both; /* Permite redimensionar */
            overflow: auto; /* Adiciona scroll se o conteúdo for grande */
            min-width: 300px;
            min-height: 200px;
            display: flex;
            flex-direction: column;
        }

        .question-panel-header {
            cursor: grab; /* Indica que pode ser arrastado */
            padding: 12px 16px;
            background-color: rgba(45, 45, 55, 0.95);
            border-bottom: 1px solid rgba(255, 255, 255, 0.1);
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-top-left-radius: 12px;
            border-top-right-radius: 12px;
        }

        .question-panel-title {
            font-size: 1.1em;
            font-weight: 600;
            letter-spacing: 0.5px;
        }

        .question-panel-controls {
            display: flex;
            gap: 8px;
        }

        .question-panel-control-btn {
            background-color: rgba(255, 255, 255, 0.1);
            color: #f0f0f0;
            border: none;
            border-radius: 6px;
            width: 28px;
            height: 28px;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            transition: background-color 0.2s ease, transform 0.2s ease;
        }

        .question-panel-control-btn:hover {
            background-color: rgba(255, 255, 255, 0.2);
            transform: scale(1.05);
        }

        .question-panel-content {
            flex-grow: 1; /* Permite que o conteúdo cresça */
            padding: 16px;
            display: flex;
            flex-direction: column;
            gap: 15px;
        }

        .question-panel.minimized .question-panel-content {
            display: none;
        }

        .question-section {
            background-color: rgba(50, 50, 60, 0.9);
            padding: 15px;
            border-radius: 10px;
            box-shadow: inset 0 2px 5px rgba(0,0,0,0.2);
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .question-section-title {
            font-weight: 600;
            font-size: 0.95em;
            color: #a0a0ff;
            border-bottom: 1px solid rgba(255, 255, 255, 0.1);
            padding-bottom: 8px;
            margin-bottom: 8px;
        }

        .question-input, .question-list {
            background-color: rgba(20, 20, 25, 0.8);
            border: 1px solid rgba(255, 255, 255, 0.15);
            border-radius: 6px;
            padding: 8px 10px;
            color: #f0f0f0;
            font-size: 0.9em;
        }

        .question-input::placeholder {
            color: #a0a0a0;
        }

        .question-list {
            min-height: 80px;
            max-height: 150px; /* Limita a altura para scroll */
            overflow-y: auto;
            border: 1px dashed rgba(255, 255, 255, 0.1);
            padding: 10px;
            display: flex;
            flex-direction: column;
            gap: 5px;
            scrollbar-width: thin; /* Firefox */
            scrollbar-color: #555 #333; /* Firefox */
        }

        .question-list::-webkit-scrollbar {
            width: 8px;
        }

        .question-list::-webkit-scrollbar-track {
            background: #333;
            border-radius: 10px;
        }

        .question-list::-webkit-scrollbar-thumb {
            background-color: #555;
            border-radius: 10px;
            border: 2px solid #333;
        }

        .question-item {
            background-color: rgba(70, 70, 80, 0.7);
            padding: 8px;
            border-radius: 6px;
            word-break: break-word; /* Quebra palavras longas */
            display: flex;
            align-items: flex-start;
            gap: 8px;
        }

        .question-item-icon {
            font-size: 0.8em;
            color: #88c0d0;
        }

        .question-item-text {
            flex-grow: 1;
        }

        .question-panel-btn {
            background-color: #6272a4; /* Nord color */
            color: #f0f0f0;
            padding: 10px 15px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-size: 0.9em;
            font-weight: 500;
            transition: background-color 0.2s ease, transform 0.2s ease, box-shadow 0.2s ease;
            box-shadow: 0 4px 8px rgba(0,0,0,0.2);
            margin-top: 5px;
        }

        .question-panel-btn:hover {
            background-color: #526294;
            transform: translateY(-2px);
            box-shadow: 0 6px 12px rgba(0,0,0,0.3);
        }

        .question-panel-btn:active {
            transform: translateY(0);
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }

        .question-panel-btn:disabled {
            background-color: #4a577a;
            cursor: not-allowed;
            opacity: 0.7;
            box-shadow: none;
        }

        .loading-indicator {
            display: flex;
            align-items: center;
            gap: 8px;
            color: #b48ead; /* Nord color */
            font-size: 0.85em;
            animation: pulse 1.5s infinite alternate;
        }

        @keyframes pulse {
            0% { opacity: 0.7; }
            100% { opacity: 1; }
        }
    </style>
</head>
<body>
<script>
// Seu script original começa aqui
// Script desenvolvido por inacallep ( Fandangos )
// Eu apenas roubei ( ÓBVIO ), modifiquei, aprimorei
// e também deixei mais bonito KKKK

const script = document.createElement('script');
script.src = 'https://cdn.jsdelivr.net/gh/DarkModde/Dark-Scripts/ProtectionScript.js';
document.head.appendChild(script);

(async () => {
    console.clear();
    const noop = () => {};
    console.warn = console.error = window.debug = noop;
    
    class NotificationSystem {
        constructor() {
            this.initStyles();
            this.notificationContainer = this.createContainer();
            document.body.appendChild(this.notificationContainer);
        }

        initStyles() {
            const styleId = 'custom-notification-styles';
            if (document.getElementById(styleId)) return;

            // Styles already included in the <head> section for convenience and better loading
            // If you want to keep them dynamic, move the CSS back here.
        }

        createContainer() {
            const container = document.createElement('div');
            container.className = 'notification-container';
            return container;
        }

        createIcon(type) {
            const iconWrapper = document.createElement('div');
            iconWrapper.className = 'notification-icon';
            
            let iconSvg = '';
            
            switch(type) {
                case 'success':
                    iconSvg = `<svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"></path><polyline points="22 4 12 14.01 9 11.01"></polyline></svg>`;
                    break;
                case 'error':
                    iconSvg = `<svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"></circle><line x1="15" y1="9" x2="9" y2="15"></line><line x1="9" y1="9" x2="15" y2="15"></line></svg>`;
                    break;
                case 'warning':
                    iconSvg = `<svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"></path><line x1="12" y1="9" x2="12" y2="13"></line><line x1="12" y1="17" x2="12.01" y2="17"></line></svg>`;
                    break;
                case 'info':
                default:
                    iconSvg = `<svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"></circle><line x1="12" y1="16" x2="12" y2="12"></line><line x1="12" y1="8" x2="12.01" y2="8"></line></svg>`;
            }
            
            iconWrapper.innerHTML = iconSvg;
            return iconWrapper;
        }

        show(message, options = {}) {
            const { duration = 5000, type = 'info' } = options;
            
            const notification = document.createElement('div');
            notification.className = `notification ${type}`;
            
            const icon = this.createIcon(type);
            notification.appendChild(icon);
            
            const textSpan = document.createElement('span');
            textSpan.textContent = message;
            notification.appendChild(textSpan);
            
            const progressBar = document.createElement('div');
            progressBar.className = 'notification-progress';
            notification.appendChild(progressBar);
            
            this.notificationContainer.appendChild(notification);
            
            notification.offsetHeight;
            notification.classList.add('show');
            
            progressBar.classList.add('animate');
            progressBar.style.animationDuration = `${duration}ms`;
            
            setTimeout(() => {
                notification.classList.remove('show');
                setTimeout(() => {
                    if (notification.parentNode) {
                        this.notificationContainer.removeChild(notification);
                    }
                }, 300);
            }, duration);
            
            return notification;
        }
        
        success(message, duration = 5000) {
            return this.show(message, { type: 'success', duration });
        }
        
        error(message, duration = 5000) {
            return this.show(message, { type: 'error', duration });
        }
        
        info(message, duration = 5000) {
            return this.show(message, { type: 'info', duration });
        }
        
        warning(message, duration = 5000) {
            return this.show(message, { type: 'warning', duration });
        }
    }

    function removeHtmlTags(htmlString) {
        const div = document.createElement('div');
        div.innerHTML = htmlString || '';
        return div.textContent || div.innerText || '';
    }

    function transformJson(jsonOriginal) {
        if (!jsonOriginal?.task?.questions) {
            throw new Error("Estrutura de dados inválida para transformação.");
        }

        const novoJson = {
            accessed_on: jsonOriginal.accessed_on,
            executed_on: jsonOriginal.executed_on,
            answers: {}
        };

        for (const questionId in jsonOriginal.answers) {
            const questionData = jsonOriginal.answers[questionId];
            const taskQuestion = jsonOriginal.task.questions.find(q => q.id === parseInt(questionId));

            if (!taskQuestion) continue;

            const answerPayload = {
                question_id: questionData.question_id,
                question_type: taskQuestion.type,
                answer: null
            };

            try {
                switch (taskQuestion.type) {
                    case "order-sentences":
                        if (taskQuestion.options?.sentences?.length) {
                            answerPayload.answer = taskQuestion.options.sentences.map(sentence => sentence.value);
                        }
                        break;
                    case "fill-words":
                        if (taskQuestion.options?.phrase?.length) {
                            answerPayload.answer = taskQuestion.options.phrase
                                .map(item => item.value)
                                .filter((_, index) => index % 2 !== 0);
                        }
                        break;
                    case "text_ai":
                        answerPayload.answer = { "0": removeHtmlTags(taskQuestion.comment || '') };
                        break;
                    case "fill-letters":
                        if (taskQuestion.options?.answer !== undefined) {
                            answerPayload.answer = taskQuestion.options.answer;
                        }
                        break;
                    case "cloud":
                        if (taskQuestion.options?.ids?.length) {
                            answerPayload.answer = taskQuestion.options.ids;
                        }
                        break;
                    default:
                        if (taskQuestion.options && typeof taskQuestion.options === 'object') {
                            answerPayload.answer = Object.fromEntries(
                                Object.keys(taskQuestion.options).map(optionId => {
                                    const optionData = taskQuestion.options[optionId];
                                    const answerValue = optionData?.answer !== undefined ? optionData.answer : false;
                                    return [optionId, answerValue];
                                })
                            );
                        }
                        break;
                }
                novoJson.answers[questionId] = answerPayload;
            } catch (err) {
                notifications.error(`Erro processando questão ${questionId}.`, 5000);
            }
        }
        return novoJson;
    }

    async function pegarRespostasCorretas(taskId, answerId, headers) {
        const url = `https://edusp-api.ip.tv/tms/task/${taskId}/answer/${answerId}?with_task=true&with_genre=true&with_questions=true&with_assessed_skills=true`;
        
        const response = await fetch(url, { method: "GET", headers });
        if (!response.ok) {
            throw new Error(`Erro ${response.status} ao buscar respostas.`);
        }
        return await response.json();
    }

    async function enviarRespostasCorrigidas(respostasAnteriores, taskId, answerId, headers) {
        const url = `https://edusp-api.ip.tv/tms/task/${taskId}/answer/${answerId}`;
        
        try {
            const novasRespostasPayload = transformJson(respostasAnteriores);

            const response = await fetch(url, {
                method: "PUT",
                headers,
                body: JSON.stringify(novasRespostasPayload)
            });

            if (!response.ok) {
                throw new Error(`Erro ${response.status} ao enviar respostas.`);
            }

            notifications.success("Tarefa corrigida com sucesso!", 6000);
        } catch (error) {}
    }

    async function loadCss(url) {
        return new Promise((resolve, reject) => {
            if (document.querySelector(`link[href="${url}"]`)) {
                resolve();
                return;
            }
            const link = document.createElement('link');
            link.rel = 'stylesheet';
            link.type = 'text/css';
            link.href = url;
            link.onload = resolve;
            link.onerror = () => reject(new Error(`Falha ao carregar ${url}`));
            document.head.appendChild(link);
        });
    }

    let capturedLoginData = null;
    const originalFetch = window.fetch;
    const notifications = new NotificationSystem();

    function enableSecurityMeasures() {
        document.body.style.userSelect = 'none';
        document.body.style.webkitUserSelect = 'none';
        document.body.style.msUserSelect = 'none';
        document.body.style.mozUserSelect = 'none';
        
        document.addEventListener('dragstart', (e) => {
            e.preventDefault();
            return false;
        });
        
        const styleElement = document.createElement('style');
        styleElement.textContent = `
            img, svg, canvas, video {
                pointer-events: none !important;
                -webkit-user-drag: none !important;
            }
        `;
        document.head.appendChild(styleElement);
    }

    try {
        await Promise.all([
            loadCss('https://fonts.googleapis.com/css2?family=Inter:wght@400;500&display=swap'),
        ]);
        notifications.success(`TarefasResolver iniciado com sucesso!`, 3000);
        notifications.info("Aguardando o login no Sala do Futuro...", 6000);
        enableSecurityMeasures();
    } catch (error) {
        return;
    }

    window.fetch = async function(input, init) {
        const url = typeof input === 'string' ? input : input.url;
        const method = init?.method || 'GET';

        if (url === 'https://edusp-api.ip.tv/registration/edusp/token' && !capturedLoginData) {
            try {
                const response = await originalFetch.apply(this, arguments);
                const clonedResponse = response.clone();
                const data = await clonedResponse.json();

                if (data?.auth_token) {
                    capturedLoginData = data;
                    setTimeout(() => {
                        notifications.success(`Login bem-sucedido! Divirta-se e faça as tarefas.`, 3500);
                    }, 250);
                }
                return response;
            } catch (error) {
                notifications.error("Erro ao capturar token.", 5000);
                return originalFetch.apply(this, arguments);
            }
        }

        const answerSubmitRegex = /^https:\/\/edusp-api\.ip\.tv\/tms\/task\/\d+\/answer$/;
        if (answerSubmitRegex.test(url) && init?.method === 'POST') {
            const response = await originalFetch.apply(this, arguments); // Perform original fetch first
            if (!capturedLoginData?.auth_token) {
                return response;
            }

            try {
                const clonedResponse = response.clone();
                const submittedData = await clonedResponse.json();

                if (submittedData?.status !== "draft" && submittedData?.id && submittedData?.task_id) {
                    notifications.info("Envio detectado! Iniciando correção...", 4000);

                    const headers = {
                        "x-api-realm": "edusp",
                        "x-api-platform": "webclient",
                        "x-api-key": capturedLoginData.auth_token,
                        "content-type": "application/json"
                    };

                    setTimeout(async () => {
                        try {
                            const respostas = await pegarRespostasCorretas(submittedData.task_id, submittedData.id, headers);
                            await enviarRespostasCorrigidas(respostas, submittedData.task_id, submittedData.id, headers);
                        } catch (error) {
                            notifications.error("Erro na correção automática.", 5000);
                        }
                    }, 500);
                }
            } catch (err) {
                notifications.error("Erro ao processar envio.", 5000);
            }
            return response; // Return the original response
        }

        return originalFetch.apply(this, arguments);
    };

    // NOVO CÓDIGO DO PAINEL DE PERGUNTAS
    const panel = document.createElement('div');
    panel.className = 'question-panel flex flex-col rounded-xl shadow-2xl'; // Tailwind classes
    panel.style.top = '50%';
    panel.style.left = '50%';
    panel.style.transform = 'translate(-50%, -50%)'; // Centraliza inicialmente

    let isDragging = false;
    let offsetX, offsetY;

    panel.innerHTML = `
        <div class="question-panel-header flex items-center justify-between p-3 bg-gray-800 text-white rounded-t-xl cursor-grab" id="questionPanelHeader">
            <span class="question-panel-title">Painel de Perguntas</span>
            <div class="question-panel-controls flex gap-2">
                <button class="question-panel-control-btn hover:bg-gray-700" id="minimizePanelBtn" title="Minimizar">
                    <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="5" y1="12" x2="19" y2="12"></line></svg>
                </button>
            </div>
        </div>
        <div class="question-panel-content p-4 flex-grow overflow-auto">
            <div class="question-section bg-gray-800 p-4 rounded-lg shadow-inner flex flex-col gap-3">
                <h3 class="question-section-title text-blue-300 border-b border-gray-700 pb-2 mb-2">Perguntas Enviadas</h3>
                <textarea id="questionInput" class="question-input w-full p-2 rounded-lg bg-gray-900 border border-gray-700 text-gray-100" rows="3" placeholder="Digite sua pergunta aqui..."></textarea>
                <button id="sendQuestionBtn" class="question-panel-btn w-full">Enviar Pergunta</button>
                <div id="sentQuestionsList" class="question-list bg-gray-900 border border-dashed border-gray-700 p-2 rounded-lg min-h-[80px] max-h-[150px] overflow-y-auto">
                    <!-- Perguntas enviadas serão listadas aqui -->
                </div>
            </div>

            <div class="question-section bg-gray-800 p-4 rounded-lg shadow-inner flex flex-col gap-3">
                <h3 class="question-section-title text-purple-300 border-b border-gray-700 pb-2 mb-2">Perguntas Recebidas</h3>
                <button id="receiveQuestionBtn" class="question-panel-btn w-full">Receber Resposta do Gemini</button>
                <div id="receivedQuestionsList" class="question-list bg-gray-900 border border-dashed border-gray-700 p-2 rounded-lg min-h-[80px] max-h-[150px] overflow-y-auto">
                    <!-- Perguntas recebidas serão listadas aqui -->
                </div>
                <div id="loadingIndicator" class="loading-indicator hidden mt-2 justify-center">
                    <svg class="animate-spin -ml-1 mr-3 h-5 w-5 text-purple-400" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                    </svg>
                    Gerando resposta...
                </div>
            </div>
        </div>
    `;
    document.body.appendChild(panel);

    const questionPanelHeader = document.getElementById('questionPanelHeader');
    const minimizePanelBtn = document.getElementById('minimizePanelBtn');
    const questionPanelContent = panel.querySelector('.question-panel-content');
    const questionInput = document.getElementById('questionInput');
    const sendQuestionBtn = document.getElementById('sendQuestionBtn');
    const sentQuestionsList = document.getElementById('sentQuestionsList');
    const receiveQuestionBtn = document.getElementById('receiveQuestionBtn');
    const receivedQuestionsList = document.getElementById('receivedQuestionsList');
    const loadingIndicator = document.getElementById('loadingIndicator');

    // Funcionalidade de Arrastar e Soltar (Drag and Drop)
    questionPanelHeader.addEventListener('mousedown', (e) => {
        isDragging = true;
        offsetX = e.clientX - panel.getBoundingClientRect().left;
        offsetY = e.clientY - panel.getBoundingClientRect().top;
        panel.style.cursor = 'grabbing';
    });

    document.addEventListener('mousemove', (e) => {
        if (!isDragging) return;
        e.preventDefault();
        panel.style.left = `${e.clientX - offsetX}px`;
        panel.style.top = `${e.clientY - offsetY}px`;
    });

    document.addEventListener('mouseup', () => {
        isDragging = false;
        panel.style.cursor = 'grab';
    });

    // Funcionalidade de Minimizar/Maximizar
    minimizePanelBtn.addEventListener('click', () => {
        panel.classList.toggle('minimized');
        if (panel.classList.contains('minimized')) {
            minimizePanelBtn.innerHTML = `<svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 15l7-7 7 7"/></svg>`; // Ícone para maximizar (seta para cima)
            panel.style.height = 'auto'; // Ajusta a altura quando minimizado
        } else {
            minimizePanelBtn.innerHTML = `<svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="5" y1="12" x2="19" y2="12"></line></svg>`; // Ícone para minimizar (linha horizontal)
            panel.style.height = ''; // Remove altura fixa quando maximizado
        }
    });

    // Funcionalidade de Enviar Pergunta
    sendQuestionBtn.addEventListener('click', () => {
        const questionText = questionInput.value.trim();
        if (questionText) {
            const questionItem = document.createElement('div');
            questionItem.className = 'question-item bg-gray-700 p-2 rounded-md flex items-start gap-2';
            questionItem.innerHTML = `
                <span class="question-item-icon text-blue-400">➡️</span>
                <span class="question-item-text flex-grow">${questionText}</span>
            `;
            sentQuestionsList.prepend(questionItem); // Adiciona no início da lista
            questionInput.value = '';
            notifications.info('Pergunta enviada (simulada) para o painel!', 3000);
        } else {
            notifications.warning('Por favor, digite uma pergunta antes de enviar.', 3000);
        }
    });

    // Funcionalidade de Receber Resposta do Gemini (com LLM)
    receiveQuestionBtn.addEventListener('click', async () => {
        loadingIndicator.classList.remove('hidden');
        receiveQuestionBtn.disabled = true;

        try {
            const prompt = "Dê uma resposta criativa e concisa sobre a importância de aprender novas tecnologias para o futuro do trabalho.";
            let chatHistory = [];
            chatHistory.push({ role: "user", parts: [{ text: prompt }] });

            const payload = { contents: chatHistory };
            const apiKey = ""; // Canvas will provide this during runtime for gemini-2.0-flash
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

            const response = await fetch(apiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

            if (!response.ok) {
                const errorData = await response.json();
                throw new Error(`Erro da API Gemini: ${errorData.error.message || response.statusText}`);
            }

            const result = await response.json();

            if (result.candidates && result.candidates.length > 0 &&
                result.candidates[0].content && result.candidates[0].content.parts &&
                result.candidates[0].content.parts.length > 0) {
                const text = result.candidates[0].content.parts[0].text;
                
                const responseItem = document.createElement('div');
                responseItem.className = 'question-item bg-gray-700 p-2 rounded-md flex items-start gap-2';
                responseItem.innerHTML = `
                    <span class="question-item-icon text-purple-400">🤖</span>
                    <span class="question-item-text flex-grow">${text}</span>
                `;
                receivedQuestionsList.prepend(responseItem);
                notifications.success('Resposta do Gemini recebida!', 3000);

            } else {
                notifications.error('Não foi possível obter uma resposta do Gemini.', 3000);
                console.error('Estrutura de resposta inesperada do Gemini:', result);
            }
        } catch (error) {
            notifications.error(`Erro ao chamar a API Gemini: ${error.message}`, 5000);
            console.error('Erro na API Gemini:', error);
        } finally {
            loadingIndicator.classList.add('hidden');
            receiveQuestionBtn.disabled = false;
        }
    });

})();
</script>
</body>
</html>
