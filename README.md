<!DOCTYPE html>
<html>
<head>
    <title>Mistura de Linguagens no Navegador</title>
    <style>
        /* CSS puro */
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            font-family: 'Comic Sans MS', cursive, sans-serif;
        }
        
        #tela {
            width: 100%;
            height: 100%;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: all 0.5s;
            font-size: 15vw;
            text-align: center;
            cursor: pointer;
        }
        
        /* CSS com variáveis JavaScript */
        .cor {
            background-color: var(--bg-color);
            color: var(--text-color);
        }
        
        .seta {
            font-size: 25vw;
            text-shadow: 0 0 20px white;
        }
    </style>
</head>
<body>
    <div id="inicio" style="width: 100%; height: 100%; display: flex; flex-direction: column; align-items: center; justify-content: center; background-color: white;">
        <h1 style="font-size: 3vw; text-align: center;">Realidade Virtual e Gamificação na Reabilitação</h1>
        <button id="botaoIniciar" style="padding: 10px 20px; font-size: 1.5vw; cursor: pointer;">Iniciar</button>
    </div>
    <div id="tela" class="cor" style="display: none;"></div>

    <!-- Carrega Pyodide (Python no navegador) -->
    <script src="https://cdn.jsdelivr.net/pyodide/v0.23.4/full/pyodide.js"></script>
    
    <script>
        const cores = [
            {nome: "VERDE", bg: "#2ECC71", text: "#FFFFFF"},
            {nome: "AZUL", bg: "#3498DB", text: "#FFFFFF"},
            {nome: "AMARELO", bg: "#F1C40F", text: "#000000"},
            {nome: "VERMELHO", bg: "#E74C3C", text: "#FFFFFF"},
            {nome: "ROSA", bg: "#FD79A8", text: "#FFFFFF"}
        ];

        const setas = ["→", "←"]; // Apenas setas para direita e esquerda
        const tela = document.getElementById('tela');
        const inicio = document.getElementById('inicio');
        const botaoIniciar = document.getElementById('botaoIniciar');
        let usandoPython = false;
        let ultimoElemento = null; // Armazena o último elemento exibido

        // Função para iniciar o jogo
        botaoIniciar.addEventListener('click', () => {
            inicio.style.display = 'none'; // Esconde a tela inicial
            tela.style.display = 'flex'; // Mostra a tela principal
            mostrarElementoJS(); // Exibe a primeira cor ou seta imediatamente
        });

        // Função em JavaScript puro
        function mostrarElementoJS() {
            let novoElemento;

            if (Math.random() < 0.85) { // 85% cores
                do {
                    novoElemento = cores[Math.floor(Math.random() * cores.length)];
                } while (novoElemento.nome === ultimoElemento); // Garante que a cor seja diferente
                tela.style.setProperty('--bg-color', novoElemento.bg);
                tela.style.setProperty('--text-color', novoElemento.text);
                tela.innerHTML = `<div>${novoElemento.nome}</div>`;
            } else { // 15% setas
                do {
                    novoElemento = setas[Math.floor(Math.random() * setas.length)];
                } while (novoElemento === ultimoElemento); // Garante que a seta seja diferente
                tela.style.setProperty('--bg-color', '#000000');
                tela.style.setProperty('--text-color', '#FFFFFF');
                tela.innerHTML = `<div class="seta">${novoElemento}</div>`;
            }

            ultimoElemento = novoElemento; // Atualiza o último elemento exibido
        }

        // Configuração do Pyodide (Python no navegador)
        async function setupPython() {
            let pyodide = await loadPyodide({
                indexURL: "https://cdn.jsdelivr.net/pyodide/v0.23.4/full/"
            });
            
            await pyodide.loadPackage("numpy");
            
            // Função em Python que será chamada pelo JS
            pyodide.runPython(`
                import numpy as np
                from js import document
                
                cores_py = [
                    {"nome": "VERDE", "bg": "#2ECC71", "text": "#FFFFFF"},
                    {"nome": "AZUL", "bg": "#3498DB", "text": "#FFFFFF"},
                    {"nome": "AMARELO", "bg": "#F1C40F", "text": "#000000"},
                    {"nome": "VERMELHO", "bg": "#E74C3C", "text": "#FFFFFF"},
                    {"nome": "ROSA", "bg": "#FD79A8", "text": "#FFFFFF"}
                ]
                
                setas_py = ["→", "←"]
                
                ultimo_elemento_py = None

                def mostrar_elemento_py():
                    global ultimo_elemento_py
                    novo_elemento = None

                    if np.random.random() < 0.85:  # 85% cores
                        while True:
                            novo_elemento = cores_py[np.random.randint(0, len(cores_py))]
                            if novo_elemento["nome"] != ultimo_elemento_py:
                                break
                        document.getElementById("tela").style.setProperty('--bg-color', novo_elemento["bg"])
                        document.getElementById("tela").style.setProperty('--text-color', novo_elemento["text"])
                        document.getElementById("tela").innerHTML = f'<div>{novo_elemento["nome"]}</div>'
                    else:  # 15% setas
                        while True:
                            novo_elemento = setas_py[np.random.randint(0, len(setas_py))]
                            if novo_elemento != ultimo_elemento_py:
                                break
                        document.getElementById("tela").style.setProperty('--bg-color', '#000000')
                        document.getElementById("tela").style.setProperty('--text-color', '#FFFFFF')
                        document.getElementById("tela").innerHTML = f'<div class="seta">{novo_elemento}</div>'

                    ultimo_elemento_py = novo_elemento  # Atualiza o último elemento exibido
            `);
            
            return pyodide;
        }

        // Alterna entre JavaScript e Python
        async function mostrarElemento() {
            if (usandoPython && window.pyodide) {
                window.pyodide.runPython("mostrar_elemento_py()");
            } else {
                mostrarElementoJS();
            }
            usandoPython = !usandoPython;
        }

        // Inicialização
        (async function() {
            try {
                window.pyodide = await setupPython();
                console.log("Python pronto! Vamos misturar JS e Python!");
            } catch (e) {
                console.log("Python não carregado, usando apenas JS");
            }
            
            // Começa ao clicar
            tela.addEventListener('click', mostrarElemento);

            // Adiciona evento para a tecla espaço
            document.addEventListener('keydown', (event) => {
                if (event.code === 'Space') { // Verifica se a tecla pressionada é espaço
                    event.preventDefault(); // Evita o comportamento padrão da tecla espaço (scroll da página)
                    mostrarElemento();
                }
            });
        })();

        // WebAssembly exemplo (somente para demonstração)
        fetch('https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@3.18.0/dist/tf.min.js')
            .then(() => console.log("TensorFlow/WebAssembly poderia ser usado aqui"))
            .catch(() => console.log("WebAssembly não carregado"));
    </script>
</body>
</html>
