# Asterisk EAGI ASR Engine (Python 3)

Este projeto é um motor de reconhecimento de voz (ASR) moderno para o Asterisk, inspirado em implementações clássicas (como as de Eder Wander), mas totalmente portado para Python 3 e otimizado para os padrões de 2026.



## 🚀 Funcionalidades

* **VAD Científico:** Utiliza **NumPy** para processamento de sinais (RMS) para detectar voz com precisão matemática diretamente no fluxo de áudio.
* **Integração EAGI:** Lê áudio bruto (L16/8kHz) do File Descriptor 3 do Asterisk em tempo real.
* **Híbrido:** Funciona tanto no servidor Asterisk quanto localmente (via microfone) para testes rápidos.
* **Multi-API:** Suporte nativo à API do Google (Chromium/Cloud) e Endpoints customizados via POST.
* **Auto-Gestão:** Instala automaticamente as dependências necessárias e gerencia variáveis de ambiente.
* **Resiliente:** Tratamento de BrokenPipe e abstração de sistema operacional (Linux/Windows).

## 🛠️ Instalação

1. Clone o repositório no seu servidor Asterisk:
```  bash
   cd /var/lib/asterisk/agi-bin/
   git clone https://github.com/programandosolucoes/asterisk-eagi-numpy-asr.git .
```


2. Dê permissões de execução:
```bash
chmod +x asr_engine.py
chown asterisk:asterisk asr_engine.py```





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

## ⚙️ Variáveis de Ambiente

O script respeita a seguinte hierarquia de configuração: **Argumentos CLI > Variáveis de Ambiente > Padrões**.

| Variável | Descrição | Padrão |
| --- | --- | --- |
| `ASR_THRESHOLD` | Sensibilidade do VAD (RMS) | 500 |
| `ASR_SILENCE` | Frames de silêncio para fechar pacote | 50 |
| `ASR_LANG` | Idioma (ex: pt-BR, en-US) | pt-BR |
| `ASR_KEY` | Chave de API Google Cloud | None |
| `ASR_ENDPOINT` | URL de API customizada | None |

## 🧪 Teste Local

Você pode testar o script no seu computador antes de subir para o Asterisk:

```bash
python3 asr_engine.py

```

Isso abrirá um menu interativo que utiliza o seu microfone local com calibração automática de ruído.

