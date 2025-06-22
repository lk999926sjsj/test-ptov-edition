(function() {
  // Cria o elemento da mensagem
  var messageDiv = document.createElement('div');
  messageDiv.textContent = 'Olá! Esta é uma mensagem do seu bookmarklet!'; // Você pode personalizar esta mensagem
  messageDiv.style.cssText = 'position: fixed; top: 20px; left: 50%; transform: translateX(-50%); background-color: rgba(0, 0, 0, 0.7); color: white; padding: 10px 20px; border-radius: 5px; z-index: 9999; font-family: sans-serif; font-size: 16px; opacity: 0; transition: opacity 0.5s ease-in-out;';
  document.body.appendChild(messageDiv);

  // Força o reflow para garantir a transição de opacidade
  messageDiv.offsetWidth;

  // Mostra a mensagem (aparecendo gradualmente)
  messageDiv.style.opacity = '1';

  // Define um tempo para a mensagem desaparecer
  setTimeout(function() {
    messageDiv.style.opacity = '0'; // Esconde a mensagem (desaparecendo gradualmente)
    // Remove o elemento da mensagem após a transição
    setTimeout(function() {
      if (messageDiv.parentNode) {
        messageDiv.parentNode.removeChild(messageDiv);
      }
    }, 500); // Deve ser igual ao tempo de transição da opacidade
  }, 3000); // Tempo que a mensagem fica visível (3 segundos)
})();
