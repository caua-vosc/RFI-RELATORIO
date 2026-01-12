const uploadForm = document.getElementById('uploadForm');
const photosInput = document.getElementById('photos');
const preview = document.getElementById('preview');
const counter = document.getElementById('counter');
const status = document.getElementById('status');

let selectedFiles = [];

function updatePreview() {
  preview.innerHTML = '';
  selectedFiles.forEach((file, index) => {
    const div = document.createElement('div');
    const img = document.createElement('img');
    img.src = URL.createObjectURL(file);
    const btn = document.createElement('button');
    btn.textContent = 'X';
    btn.className = 'remove-btn';
    btn.onclick = () => {
      selectedFiles.splice(index, 1);
      updatePreview();
    };
    div.appendChild(img);
    div.appendChild(btn);
    preview.appendChild(div);
  });
  counter.textContent = selectedFiles.length;
}

// Atualiza arquivos selecionados
photosInput.addEventListener('change', (e) => {
  const files = Array.from(e.target.files);
  selectedFiles = selectedFiles.concat(files).slice(0, 10); // Limite 10
  updatePreview();
  photosInput.value = ''; // Limpa input para permitir reupload
});

// Envio do formulário
uploadForm.addEventListener('submit', async (e) => {
  e.preventDefault();
  if (selectedFiles.length === 0) {
    alert('Selecione pelo menos uma foto!');
    return;
  }

  const siteId = document.getElementById('siteId').value.trim();
  const section = document.getElementById('section').value;

  if (!siteId || !section) {
    alert('Informe o ID do site e a seção.');
    return;
  }

  const formData = new FormData();
  formData.append('siteId', siteId);
  formData.append('section', section);
  selectedFiles.forEach(file => formData.append('photos', file));

  status.textContent = 'Enviando...';

  try {
    const BACKEND_URL = 'https://backend-3-frp3.onrender.com';
    const res = await fetch(BACKEND_URL, {
      method: 'POST',
      body: formData
    });

    const data = await res.json();
    if (data.success) {
      status.textContent = `Upload concluído: ${data.files.join(', ')}`;
      selectedFiles = [];
      updatePreview();
    } else {
      status.textContent = 'Erro: ' + (data.error || 'Falha desconhecida');
    }
  } catch (err) {
    status.textContent = 'Erro ao enviar: ' + err.message;
  }
});
