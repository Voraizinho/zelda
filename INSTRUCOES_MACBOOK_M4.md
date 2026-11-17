# Guia de Instalação e Execução no MacBook (Apple Silicon M4)
> **Destinado ao Agente de IA e Desenvolvedores no macOS**

Este repositório contém o código, scripts de extração, tradutor de shaders e runtime do **Zelda: Ocarina of Time Remake (Recompilação Nativa)**.

---

## 🚀 Por que o MacBook M4 roda muito melhor?
1. **Instruções ARM64 Nativas:** O código do jogo original do Switch é binário ARM64. No chip M4 da Apple, a CPU executa diretamente sem sobrecarga de tradução para x86-64.
2. **Suporte de Hardware ASTC:** A GPU do M4 decodifica texturas ASTC nativamente no silício. Na GPU de PC (NVIDIA GeForce), é necessário converter para BC3 via compute shader; no Mac, isso é feito a **custo zero de processamento e máxima qualidade gráfica**.
3. **Memória Unificada (Zero-Copy):** A arquitetura do M4 compartilha memória RAM e VRAM em alta largura de banda (120+ GB/s), eliminando gargalos de VRAM de 6 GB e tráfego PCIe.

---

## 🛠️ 1. Pré-requisitos no macOS

Execute no terminal do Mac (usando Homebrew):

```bash
# 1. Ferramentas essenciais de compilação
xcode-select --install
brew install git cmake ninja pkg-config clang-format
brew install sdl3 opus dav1d libvpx zstd
brew install dotnet-sdk

# 2. Gerenciador rápido de Python (uv)
brew install uv

# 3. Suporte Vulkan / MoltenVK (para o backend Vulkan)
brew install --cask vulkan-sdk moltenvk
```

---

## 🐍 2. Configurar os Ambientes Python

A partir da raiz do repositório:

```bash
# Ambiente de build e lifter (Python 3.12)
uv venv --python 3.12 .venv
uv pip install --python .venv/bin/python -r tools/requirements.txt

# Ambiente de extração (requer Python 3.14 com zstd nativo)
uv venv --python 3.14 .venv-py314
uv pip install --python .venv-py314/bin/python -r tools/requirements-py314.txt
```

---

## 🎮 3. Configurar sua Cópia do Jogo e Keys

1. Copie o arquivo `config.example.toml` para `config.local.toml`:
   ```bash
   cp config.example.toml config.local.toml
   ```
2. Edite o arquivo `config.local.toml` indicando o caminho do seu jogo e da sua chave:
   ```toml
   [paths]
   game = "/caminho/para/seu-jogo.nup"      # ou .nsp / .xci
   keys = "/caminho/para/prod.keys"
   ```

---

## ⚙️ 4. Extrair e Compilar

Execute os scripts automatizados de compilação:

```bash
# Permissão de execução para os scripts bash
chmod +x compilar.sh jogar/jogar-ocarina.sh

# Compilar automaticamente
./compilar.sh --jogo /caminho/para/seu-jogo.nup --sem-perguntar
```

Ou execute as etapas manualmente:

```bash
# A. Compilar hactool com patches de compatibilidade
.venv/bin/python tools/build_hactool.py

# B. Extrair o jogo e mesclar atualizações RomFS
.venv-py314/bin/python tools/extract.py

# C. Compilar o tradutor de shaders (.NET NativeAOT)
.venv/bin/python tools/build_xlate.py

# D. Gerar o código C, patches de taxa de quadros e compilar o executável
.venv/bin/python recomp/gen.py --exefs build/extracted/v111/exefs --out generated/oot111 --patches recomp/patches-v111.txt
.venv-py314/bin/python tools/make_romfs_patches.py --version v111
.venv/bin/python tools/build_game.py --version v111
```

---

## ⚡ 5. Otimizações Recomendadas para o M4 (`totk.ini`)

No arquivo `totk.ini` gerado na pasta do jogo:

```ini
[display]
fullscreen = 1
output_res = 2560x1440       ; ou resolução nativa da tela Retina
render_scale = 100           ; O M4 suporta 100% nativo com folga
upscaler = fsr
sharpness = 85
vk_present = fifo            ; V-Sync perfeito sincronizado com ProMotion (60/120Hz)
perf_overlay = fps

[frame rate]
fps_limit = 60
fps60 = 1
realtime = 1
cloth_realtime = 1
highfps_fix = 1

[performance]
astc_bc = 0                  ; 0 = Deixa a GPU da Apple decodificar ASTC por hardware (qualidade pura)
vk_dyn_a2c = 0               ; Evita bug de silhueta preta
timer_1ms = 1
lib_prebuild = 1
pipe_prebuild = 1
```

---

## 🕹️ 6. Jogar

Para iniciar:
```bash
./jogar/jogar-ocarina.sh
```

Aproveite Zelda com o poder nativo do Apple Silicon M4!
