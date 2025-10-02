Configuração do Argent One + FlexASIO + RS_ASIO

Este repositório fornece configuração e instruções para usar a interface de áudio **Armer Argent One** com **FlexASIO** (driver ASIO universal) e **RS_ASIO** (patch Rocksmith 2014 para habilitar ASIO). A configuração permite executar Guitar Rig, Rocksmith e áudio padrão do Windows (YouTube, Discord, etc.) simultaneamente sem conflitos.

* [RS_ASIO](https://github.com/mdias/rs_asio)
* [FlexASIO](https://github.com/dechamps/FlexASIO)
* [FlexASIO GUI](https://github.com/flipswitchingmonkey/FlexASIO_GUI)

---

## Índice

1. [Visão geral](#visão-geral)
2. [Pré-requisitos](#pré-requisitos)
3. [Arquivos de configuração incluídos](#arquivos-de-configuração-incluídos)
4. [Instruções de configuração](#instruções-de-configuração)
5. [Teste e ajuste de latência](#teste--ajuste-de-latência)
6. [Streaming e bypass ASIO](#streaming--asio-bypass)
7. [Solução de problemas](#troubleshooting)
8. [Detalhes do hardware Argent One](#argent-one-hardware-details)
9. [Créditos e referências](#credits--references)

---

## Visão geral

* **Argent One** é uma interface de áudio da Armer.
* **FlexASIO** funciona como um driver ASIO universal, fazendo a ponte entre hosts ASIO e backends de áudio do Windows (WASAPI, etc.), permitindo que vários aplicativos compartilhem a mesma interface.
* **RS_ASIO** injeta suporte ASIO no **Rocksmith 2014**, substituindo ou aumentando seu tratamento de áudio WASAPI.

Com essa configuração, é possível:

* Usar o Guitar Rig (ou outros simuladores VST/amp) via ASIO através do FlexASIO
* Jogar Rocksmith com ASIO (via RS_ASIO)
* Permitir que o áudio do Windows (por exemplo, navegador, sons do sistema) coexista com aplicativos ASIO

---

## Pré-requisitos

* PC com Windows
* Interface **Armer Argent One**, instalada e funcional
* **FlexASIO** instalado
* **FlexASIO GUI** (opcional, mas útil)
* **RS_ASIO** aplicado ao Rocksmith 2014
* Capacidade de editar arquivos `.toml` e `.ini`
* .NET Desktop Runtime 6.x instalado (para a GUI)

---

## Arquivos de configuração incluídos

* `FlexASIO.toml` — Configuração do FlexASIO
* `RS_ASIO.ini` — Configuração do RS_ASIO

### FlexASIO.toml

```toml
backend = “Windows WASAPI”
# sampleRate = 48000     # A GUI pode travar se não for comentada
bufferSizeSamples = 512

[input]
device = “Microfone (2- Armer Argent)”
channels = 1
wasapiExclusiveMode = false
wasapiAutoConvert = true

[output]
device = “Fones de ouvido (2- Armer Argent)”
wasapiExclusiveMode = false
wasapiAutoConvert = true
```

> A GUI do FlexASIO **não** suporta o campo `sampleRate`. Deixe-o comentado ou remova-o completamente.

### RS_ASIO.ini

```ini
[Config]
EnableWasapiOutputs = 0
EnableWasapiInputs = 0
EnableAsio = 1

[Asio]
BufferSizeMode = driver

[Asio.Output]
Driver = FlexASIO
BaseChannel = 0
EnableSoftwareEndpointVolumeControl = 0
EnableSoftwareMasterVolumeControl = 0

[Asio.Input.0]
Driver = FlexASIO
Channel = 0
EnableSoftwareEndpointVolumeControl = 0
EnableSoftwareMasterVolumeControl = 0

[Asio.Input.1]
Driver =
Canal =
EnableSoftwareEndpointVolumeControl = 0
EnableSoftwareMasterVolumeControl = 0
```

---

## Instruções de configuração

1. **Instale o FlexASIO**
   Faça o download a partir dos [lançamentos oficiais](https://github.com/dechamps/FlexASIO/releases/).
   Execute o instalador para que o `FlexASIO.dll` e os arquivos relacionados sejam instalados.

2. **Instale o RS_ASIO**
   Faça o download nas [versões do RS_ASIO](https://github.com/mdias/rs_asio/releases/).
   Copie o `RS_ASIO.dll`, o `RS_ASIO.ini` e os arquivos relacionados para a pasta do jogo Rocksmith.

3. **Coloque os arquivos de configuração**

* Copie o `FlexASIO.toml` para o diretório do perfil do usuário (por exemplo, `C:\Users\<SeuNome>\FlexASIO.toml`)
* Substitua o `RS_ASIO.ini` na pasta Rocksmith pelo arquivo deste repositório

4. **Configure o áudio do Windows**

   * Defina o Argent One como o dispositivo padrão de reprodução e captura
   * Selecione 48 kHz nas propriedades do dispositivo. Essa é a taxa recomendada para o Rocksmith; outros aplicativos podem usar taxas de amostragem diferentes, se necessário
   * Desative os aprimoramentos de áudio e o controle exclusivo

5. **Inicie a GUI do FlexASIO (opcional)**

   * Certifique-se de que `sampleRate` esteja comentado ou removido para evitar falhas
   * Use a GUI para confirmar a seleção do dispositivo e o tamanho do buffer

6. **Inicie o Rocksmith**

   * O RS_ASIO se conectará ao jogo, detectará o FlexASIO e usará a configuração fornecida

7. **Inicie o Guitar Rig / outros aplicativos ASIO**

   * Selecione `FlexASIO` como o driver ASIO
   * Ambos os aplicativos compartilharão a interface Argent One

8. **Toque e monitore**
   O áudio da guitarra, do jogo e do sistema agora devem funcionar simultaneamente.

---

## Teste e ajuste de latência

* Comece com `bufferSizeSamples = 512` (equilibrado)
* Se ocorrerem estalos/pops, aumente para `1024` ou mais
* Para uma latência mais baixa, experimente com `256` ou `128`
* Revise `RS_ASIO-log.txt` para verificar as configurações do buffer e verificar se há xruns
* Monitore a latência da CPU/DPC com ferramentas como LatencyMon
* Certifique-se de que todos os dispositivos estejam configurados em 48 kHz para evitar sobrecarga de reamostragem

---

## Streaming e bypass ASIO

Quando o RS_ASIO lida com a saída de áudio do Rocksmith através do ASIO, a pilha de áudio do Windows é ignorada. Isso impede a captura por software de streaming/gravação. As opções incluem:

* Usar a saída WASAPI no RS_ASIO (em vez do ASIO) para que o áudio do jogo passe pelo mixer do Windows
* Empregar ferramentas de roteamento de áudio virtual (por exemplo, VB-Audio Cable, VoiceMeeter)
* Para configurações simples de streaming, habilitar a saída WASAPI é a solução mais direta

---

## Solução de problemas

| Sintoma                 | Causa provável                        | Correção sugerida                                          |
| ----------------------- | ------------------------ ----------- | ------------------------------------------------------ |
| A GUI trava ao iniciar    | `sampleRate` presente no TOML        | Remova ou comente `sampleRate`                     |
| Rocksmith sem som        | incompatibilidade ou configuração incorreta do driver | Verifique `RS_ASIO-log.txt` e verifique `Driver = FlexASIO` |
| Estalos/falhas     | buffer muito pequeno, problemas com USB/alimentação  | Aumente o buffer, desative a economia de energia USB              |
| Latência muito alta        | Buffer muito grande                    | Reduza o buffer gradualmente                                |
| Sem áudio do jogo no stream | ASIO ignorando o Windows              | Habilite a saída WASAPI ou use o roteamento de áudio virtual      |
| GUI não mostra dispositivos    | Dispositivo desativado nas configurações do Windows | Habilite o dispositivo nas configurações de som do Windows                |

---

## Detalhes do hardware Argent One

* [Página do produto](https://armer.com.br/produtos/interface-de-audio-usb-armer-argent-one/)
* [Manual oficial (PDF)](https://drive.google.com/file/d/15iVYalthiWtdguu4y76YLcfhcIaOtVLd/view?usp=sharing)

### Principais recursos

* Duas entradas combinadas (XLR / TRS de ¼”).
* Alimentação fantasma de +48 V para microfones condensadores (Canal 1).
* Canal 2 selecionável entre **Instrumento (INST)** e **Linha**.
* Interruptor de monitoramento direto (MON) para monitoramento sem latência.
* Saídas TRS balanceadas para monitores de estúdio.
* Saída dedicada para fones de ouvido com controle de ganho próprio.

### Observações importantes sobre o comportamento

**A) Apenas microfone (Canal 1)**

* Funciona como uma entrada de microfone mono.
* Tipo de conector: **XLR de 3 pinos, mono balanceado**. Balanceado aqui significa que ele transporta o mesmo sinal mono em dois fios com polaridade oposta para cancelar o ruído, não que seja estéreo. Os novos usuários costumam confundir “balanceado” com “estéreo”.
* A alimentação fantasma de +48 V pode ser ativada se você estiver usando um microfone condensador.
* Com o **Monitor Direto ativado**, você ouvirá o microfone em mono com latência zero. Se o monitoramento DAW também estiver ativado, você poderá ouvir um eco devido ao sinal duplicado atrasado.

**B) Instrumento (Ch2) com INST ON**

* Entrada não balanceada de alta impedância (Hi-Z) para guitarra, baixo, etc.
* Tipo de conector: **¼” TS (2 pólos), mono não balanceado**. Os cabos TS transportam um sinal mono, não estéreo. Os novos usuários costumam confundir TS com conectores estéreo, mas ele transporta apenas um único canal.
* Tratado como uma fonte mono.
* **Monitor direto ativado** reproduz em mono instantaneamente; desative o monitoramento DAW para evitar ouvir duas vezes.

**C) Fonte de nível de linha (Ch2 com INST desativado)**

* Entrada TRS mono balanceada, projetada para teclados, sintetizadores ou saídas de mixer.
* Tipo de conector: **TRS de ¼” (3 pólos), mono balanceado**. Balanceado significa que o mesmo sinal mono é transportado em dois condutores com polaridade oposta para rejeitar ruídos. Não é estéreo — novos usuários costumam confundir TRS balanceado com TRS estéreo.
* Funciona como uma fonte mono.
* **Monitor direto ativado** reproduz em mono com latência zero.

**D) Microfone (Canal 1) + Instrumento/Linha (Canal 2) simultaneamente**

* **Observação manual:** se as entradas de microfone e instrumento forem usadas ao mesmo tempo, **ambas as entradas estarão em mono**.
* **Monitor direto ativado:** o sinal monitorado é uma **mixagem mono** de ambas as entradas com latência zero.
* **Monitor direto desligado:** você só ouve o que o DAW retorna.
* **Gravação no DAW:** o manual não confirma explicitamente se o computador os recebe como canais separados ou como um feed mono somado. Na prática, a maioria das interfaces 2×2 compatíveis com a classe ainda apresentam **Entrada 1** e **Entrada 2** separadamente; verifique a lista de entradas do seu DAW para confirmar.

**Saída de fone de ouvido e contexto de monitoramento direto**

* O conector de fone de ouvido suporta TRS estéreo padrão; fones de ouvido de 4 pólos podem não funcionar corretamente.
* O monitor direto afeta o que você ouve nos fones de ouvido e nas saídas de linha: ele ignora o computador e mixa as entradas diretamente nas saídas, sempre em mono.

**Alimentação fantasma**

* Fornece 48 V apenas para o Canal 1. Use com microfones condensadores que exijam isso.
* Desative a alimentação fantasma quando não estiver em uso para evitar ruídos indesejados ou possíveis danos aos microfones dinâmicos/de fita.

---

## Créditos e referências

* **RS_ASIO** — patch para suporte ASIO do Rocksmith 2014
* **FlexASIO** — driver ASIO universal usando PortAudio e backends de áudio do Windows
* **FlexASIO GUI** — GUI auxiliar para editar `FlexASIO.toml`
* **Armer Argent One** — detalhes de hardware com base no [manual oficial] e nas especificações do fabricante
