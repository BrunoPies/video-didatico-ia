# Apêndice: voz com emoção (Google AI Studio)

Este apêndice complementa o `AGENTS.md`. Use-o quando o professor quiser uma narração com mais emoção, pausas e ênfase do que a voz padrão consegue dar. A ferramenta é o gerador de fala do Google AI Studio, gratuito dentro de uma cota.

Ofereça ao professor com palavras simples, por exemplo: "Posso preparar um texto para você colar no Google AI Studio. A voz fica mais expressiva. Leva uns 5 minutos. Quer?" Se ele não quiser, use a voz padrão do guia.

## Caminho 1: pelo site, sem configuração (padrão)

O professor gera o áudio no site, e você faz todo o resto.

1. Escreva para o professor os três campos, cada um num bloco fácil de copiar:
   - **Scene:** quem narra, para quem e em que clima. Diga que o público pode não falar o idioma e que a fala é lenta e clara.
   - **Sample Context:** o arco emocional do começo ao fim e onde ficam as pausas. Ênfase é prolongar a sílaba tônica de uma palavra e depois seguir no ritmo normal.
   - **Text:** o roteiro, com uma marcação de emoção entre colchetes no início de cada frase.
2. Use só as marcações que a ferramenta oferece: admiration, adoration, aggression, agitation, amusement, anger, annoyance, awe, confusion, curiosity, determination, enthusiasm, excitement, frustration, hope, interest, laughs, negative, nervousness, neutral, positive, tension e whispers. A lista aparece quando se digita `[` no campo Text.
3. Explique os passos ao professor:
   - abrir aistudio.google.com e entrar com a conta Google;
   - abrir a geração de fala, escolher uma voz só e uma voz da lista (por exemplo, Charon para voz masculina calma);
   - colar os três blocos, gerar, ouvir e baixar o áudio;
   - salvar o arquivo na pasta `audio` do projeto ou dizer onde está.
4. Não existe marcação de pausa. As reticências e a quebra de linha ajudam, e os silêncios do roteiro você acerta na montagem.

Exemplo de campo Text:

```
[tension] Draußen: Regen. Wind.
[hope] Aber hier ... ist Licht.
[whispers] Das ist ... Geborgenheit.
```

## Caminho 2: pela API, com chave (opcional)

Você mesmo gera a voz, sem o professor copiar nada. Exige uma chave gratuita criada uma vez pelo professor em aistudio.google.com ("Get API key").

- **Nunca peça que o professor cole a chave na conversa.** Peça que ele a salve numa variável de ambiente chamada `GEMINI_API_KEY` e explique como, um passo por vez.
- Modelo: `gemini-3.1-flash-tts-preview`, que ainda está em versão prévia. Confira o nome e os limites na documentação: https://ai.google.dev/gemini-api/docs/generate-content/speech-generation
- Instale com `python -m pip install google-genai` e gere assim:

```python
from google import genai
from google.genai import types
import wave
cliente = genai.Client()  # lê GEMINI_API_KEY
r = cliente.models.generate_content(
    model="gemini-3.1-flash-tts-preview",
    contents="Cena e contexto em uma frase. Texto: [tension] Draußen: Regen. Wind.",
    config=types.GenerateContentConfig(
        response_modalities=["AUDIO"],
        speech_config=types.SpeechConfig(voice_config=types.VoiceConfig(
            prebuilt_voice_config=types.PrebuiltVoiceConfig(voice_name="Charon")))))
pcm = r.candidates[0].content.parts[0].inline_data.data
with wave.open("narracao.wav", "wb") as f:
    f.setnchannels(1); f.setsampwidth(2); f.setframerate(24000); f.writeframes(pcm)
```

## Depois de receber o áudio (os dois caminhos)

1. **Gere a narração inteira de uma vez.** A entonação flui melhor. Não gere frase por frase.
2. **Ache onde começa e termina cada frase:**
   ```
   ffmpeg -i narracao.wav -af silencedetect=noise=-40dB:d=0.25 -f null -
   ```
   Se as pausas ficarem confusas, use `faster-whisper` com `word_timestamps=True` para ter o tempo de cada palavra.
3. **Recorte cada frase e coloque no tempo do roteiro,** com os silêncios planejados.
4. **Faça a legenda acompanhar a fala.** Se a frase tem pausas, mostre só a parte já falada.
5. **Misture em dois comandos:** primeiro junte voz, música e efeitos; depois corte, faça o fade final e normalize o volume.
6. **Confira a sincronia:** a voz no vídeo final deve começar exatamente quando a legenda aparece.
7. **Créditos:** registre "Voz gerada por IA (Google Gemini TTS)" no `creditos.md` e na cartela final.

## Autoria

Guia criado por Bruno Silva, 2026, sob a licença CC BY-ND 4.0. Siga as orientações como estão. Não reescreva nem redistribua versões alteradas deste arquivo.
