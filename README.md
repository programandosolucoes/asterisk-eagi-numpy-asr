Este projeto é um motor de reconhecimento de voz (ASR) moderno para o Asterisk, inspirado em implementações clássicas (como as de Eder Wander), mas totalmente portado para Python 3 e otimizado para os padrões de 2026.

## 📌 Versão e Compatibilidade

* **Versão Atual:** 1.0.0 (Release Absoluta 2026)
* **Compatibilidade:** Python 3.8+
* **Sistemas:** Linux (Produção EAGI), Windows/macOS (Testes Locais)

---

## 🚀 Funcionalidades

* **VAD Científico:** Utiliza **NumPy** para processamento de sinais (RMS) para detectar voz com precisão matemática diretamente no fluxo de áudio.
* **Integração EAGI:** Lê áudio bruto (PCM Linear 16-bit, 8kHz) do File Descriptor 3 do Asterisk em tempo real.
* **Híbrido:** Funciona tanto no servidor Asterisk quanto localmente (via microfone) para testes rápidos.
* **Multi-API:** Suporte nativo à API do Google (Chromium/Cloud) e Endpoints customizados via POST.
* **Auto-Gestão:** Instala automaticamente as dependências Python necessárias e gerencia variáveis de ambiente.
* **Resiliente:** Tratamento de BrokenPipe e abstração de sistema operacional (Linux/Windows).

---

## 🧠 Como Funciona (Explicação Técnica)

O motor opera em um ciclo de 4 etapas para garantir baixa latência:

1. **Captura:** O Asterisk via EAGI espelha o áudio da chamada para o `File Descriptor 3`. O script lê blocos de 320 bytes (20ms).
2. **VAD via NumPy:** O script calcula o valor **RMS (Root Mean Square)** de cada bloco. A voz só é considerada "ativa" se a energia sonora ultrapassar o `THRESHOLD`.
3. **Buffer Dinâmico:** O áudio é acumulado em memória (`io.BytesIO`). O script monitora o silêncio; se durar mais que o definido em `SILENCE`, o buffer é selado.
4. **Reconhecimento:** O áudio é enviado à API. O resultado textual retorna ao Asterisk através da variável `${ASR_RESULT}` via protocolo AGI.

---

## 📦 Dependencies

O sistema requer as seguintes bibliotecas de manipulação de áudio instaladas no S.O.:

* **flac** >= 1.2.1
* **libflac-dev** >= 1.2.1
* **libsndfile** >= 1.0.21
* **libsndfile-dev** >= 1.0.21

### Instalação das Dependências (Ubuntu/Debian)

```bash
sudo apt-get update
sudo apt-get install flac libflac-dev libsndfile1 libsndfile1-dev python3-pip

```

---

## 🛠️ Instalação

1. **Clone o repositório no seu servidor Asterisk:**

```bash
cd /var/lib/asterisk/agi-bin/
git clone https://github.com/programandosolucoes/asterisk-eagi-numpy-asr.git .

```

2. **Dê permissões de execução:**

```bash
chmod +x asr_engine.py
chown asterisk:asterisk asr_engine.py

```

3. **Provisionamento de Bibliotecas Python:**

```bash
pip3 install numpy SpeechRecognition requests

```

---

## 📞 Uso no Dialplan (extensions.conf)

Chame o script usando a aplicação `EAGI`. Você pode passar argumentos para customizar o comportamento:

```asterisk
exten => 100,1,Answer()
 same => n,Playback(aguarde-o-sinal)
 ; Executa com threshold de 600 e idioma pt-BR
 same => n,EAGI(asr_engine.py, --threshold 600 --lang pt-BR)
 same => n,NoOp(Resultado do ASR: ${ASR_RESULT})
 same => n,SayText(Você disse: ${ASR_RESULT})
 same => n,Hangup()

```

---

## ⚙️ Variáveis de Ambiente

O script respeita a seguinte hierarquia de configuração: **Argumentos CLI > Variáveis de Ambiente > Padrões**.

| Variável | Argumento CLI | Descrição | Padrão |
| --- | --- | --- | --- |
| `ASR_THRESHOLD` | `-t`, `--threshold` | Sensibilidade do VAD (RMS) | 500 |
| `ASR_SILENCE` | `-s`, `--silence` | Chunks de silêncio para fechar | 50 |
| `ASR_LANG` | `-l`, `--lang` | Idioma (ex: pt-BR, en-US) | pt-BR |
| `ASR_KEY` | `--key` | Chave de API Google Cloud | None |
| `ASR_ENDPOINT` | `--endpoint` | URL de API customizada | None |

---

## 🧪 Teste Local

Você pode testar o script no seu computador antes de subir para o Asterisk:

```bash
python3 asr_engine.py

```

Isso abrirá um menu interativo que utiliza o seu microfone local com calibração automática de ruído.

---

**Nota Histórica:** Este projeto é uma evolução das implementações de 2012, atualizado para ser "zero-touch" e resiliente em infraestruturas modernas de telecomunicações.
