# Instalação — Fase 0: Home Assistant no Mac

Objetivo desta fase: ter o Home Assistant a correr numa máquina virtual no MacBook Pro M1, com os dispositivos ligados diretamente ao HA (sem a app SmartLife) e com nomes em português. Demora cerca de 1 hora.

> **Porquê uma VM e não Docker?** No macOS o Docker corre dentro de uma VM própria e não anuncia serviços na rede local (mDNS). O HomeKit Bridge (Siri) e a descoberta de dispositivos precisam disso. Uma VM do UTM em modo *bridge* comporta-se como um computador verdadeiro na rede. Quando chegar o mini-PC, fazes um backup aqui e restauras lá.

## 1. Preparar o Mac

1. **Não deixar o Mac adormecer** quando é o servidor: Definições do Sistema → Bateria → Opções → ativa "Impedir a suspensão automática no transformador quando o ecrã está desligado". Mantém a tampa aberta (o ecrã pode desligar-se) e o carregador ligado.
2. **IP fixo para o HA:** depois do passo 2, reserva no router o IP da VM (procura "DHCP reservation" ou "IP estático" nas definições do router).

## 2. Instalar o Home Assistant OS no UTM

1. Instala o **UTM**: <https://mac.getutm.app> (grátis).
2. Descarrega o HA OS para ARM: em <https://github.com/home-assistant/operating-system/releases/latest> descarrega o ficheiro `haos_generic-aarch64-<versão>.qcow2.xz`. Faz duplo clique para descomprimir e fica com um `.qcow2`.
3. No UTM: **Criar nova máquina virtual → Virtualizar → Outro**.
   - Arranque de ISO: marca **Ignorar ISO**.
   - Memória: **4096 MB**. Núcleos: **2**.
   - Armazenamento: aceita o valor por omissão (vamos apagar este disco a seguir).
   - Nome: `Home Assistant`. Marca **Abrir definições da VM**.
4. Nas definições da VM:
   - **Rede:** modo **Bridged (Avançado)**, interface `en0` (Wi-Fi). Se o Mac estiver ligado por cabo, escolhe a interface Ethernet.
   - **Unidades:** apaga o disco que o UTM criou. Clica em **Nova → Importar** e escolhe o `.qcow2`. Interface: **VirtIO**. Depois seleciona o disco e usa **Redimensionar** para **32 GB**.
   - **Sistema / QEMU:** confirma que o **arranque UEFI** está ativo (vem ativo por omissão).
5. Guarda e arranca a VM. Espera 3 a 5 minutos.
6. No Safari, abre <http://homeassistant.local:8123>. Se não abrir, vê o IP que aparece na consola da VM e usa `http://<IP>:8123`.

## 3. Configuração inicial do HA

1. Cria o teu utilizador (vais usá-lo para tudo).
2. Localização: põe a tua casa no mapa. Fica guardada só no HA, não no repositório.
3. Idioma **Português (Portugal)**, fuso **Europe/Lisbon**, unidades **métricas**, moeda **EUR**.
4. Em **Definições → Sistema → Cópias de segurança**, ativa as cópias de segurança automáticas diárias.

## 4. Add-ons de base

Em **Definições → Suplementos → Loja de suplementos**, instala:

- **Studio Code Server** (para editar ficheiros do HA no browser). Ativa "Mostrar na barra lateral".
- **Terminal & SSH** (só se precisares do HACS no passo 6).

## 5. Dispositivos Zigbee: diretamente no HA, sem conta Tuya (caminho principal)

Os dispositivos **Zigbee** passam a falar com um adaptador Zigbee teu, ligado ao HA. Não precisas de conta Tuya, de nuvem nem da app SmartLife: o HA (através do Zigbee2MQTT) encontra-os diretamente.

**Precisas do adaptador.** Para quem usa VM, recomendo o **SLZB-06 (Ethernet/PoE)** em vez de uma pen USB. Liga-se pela rede, por isso não há passagem de USB para a VM e a mudança para o mini-PC é trivial. A pen SONOFF ZBDongle-E também funciona: no UTM, liga-a à VM pelo ícone de USB da barra de ferramentas.

1. **Suplementos → Loja → Mosquitto broker**: instala e arranca.
2. **Loja → ⋮ → Repositórios**: adiciona `https://github.com/zigbee2mqtt/hassio-zigbee2mqtt` e instala o **Zigbee2MQTT**.
3. Configuração do Zigbee2MQTT (separador Configuração, secção `serial`):
   - SLZB-06: `port: tcp://<IP-do-SLZB-06>:6638` e `adapter: ember` (firmware EFR32) ou `zstack` (firmware CC2652). Vê qual tens na página web do SLZB-06.
   - ZBDongle-E: `port: /dev/serial/by-id/<o-que-aparecer>` e `adapter: ember`.
4. Arranca o Zigbee2MQTT e abre-o pela barra lateral.
5. **Para cada dispositivo Zigbee**, um de cada vez:
   1. Na SmartLife, remove o dispositivo do gateway ("Remover dispositivo"). É a última vez que usas a SmartLife para ele.
   2. No Zigbee2MQTT, carrega em **Permitir adesão (Permit join)**.
   3. Põe o dispositivo em modo de emparelhamento. Normalmente é manter o botão premido 5 a 10 segundos até a luz piscar depressa. Nas persianas costuma ser o botão de parar.
   4. Quando aparecer, dá-lhe o nome final (ver `docs/DEVICES.md`).
6. Quando já não houver nada no gateway Tuya, podes desligá-lo.

## 6. Dispositivos Wi-Fi Tuya (só se tiveres algum)

Os dispositivos Tuya **Wi-Fi** cifram a comunicação com uma chave (*local key*) que só a Tuya conhece. Por isso, para os controlar localmente é preciso ir buscá-la **uma única vez**. Depois disso o HA fala com eles diretamente na rede de casa, sem nuvem. Há duas formas de o fazer:

**A. Buscar a chave uma vez com a conta SmartLife que já tens** (não é preciso criar conta nova)

1. **Instalar o HACS:** abre o Terminal (barra lateral) e corre:
   ```
   wget -O - https://get.hacs.xyz | bash -
   ```
   Reinicia o HA (**Definições → Sistema → Reiniciar**). Depois vai a **Adicionar integração → HACS** e segue o pedido de autorização do GitHub.
2. No HACS, procura e instala **Tuya Local** (de `make-all`). Reinicia o HA.
3. **Adicionar integração → Tuya Local** e escolhe a configuração **assistida pela nuvem (Smart Life)**. Na app SmartLife vai a **Eu → ⚙️ → Conta e segurança → Código de utilizador**, cola o código e lê o QR com a SmartLife.
4. Adiciona cada dispositivo Wi-Fi e escolhe o tipo sugerido. Daqui para a frente funciona sem nuvem: se quiseres, podes bloquear o acesso à internet desses dispositivos no router.

> Se voltares a emparelhar um destes dispositivos na SmartLife, a chave muda e tens de repetir o passo 3 para ele.

**B. Substituir por equivalentes Zigbee**

Se não quiseres tocar na Tuya nem uma vez, troca os dispositivos Wi-Fi por versões Zigbee (uma lâmpada Zigbee com temperatura de cor custa 10 a 20 €) e usa o passo 5.

> Não uses a integração **Tuya** oficial do HA: depende sempre da nuvem da Tuya.

## 7. Token para a app (prepara já, usa-se na Fase 1)

1. No HA, clica no teu nome (canto inferior esquerdo) → **Segurança → Tokens de acesso de longa duração → Criar token**. Nome: `dashboard`.
2. Copia o token **e guarda-o no gestor de palavras-passe**. O HA só o mostra uma vez.
3. Nunca o ponhas no Git nem no código da app. Na Fase 1 fica no ficheiro `.env` do servidor, e quem o usa é o Caddy, não o browser.

## 8. Copiar a configuração deste repositório para o HA

1. Abre o **Studio Code Server**.
2. Copia `homeassistant/configuration.yaml` deste repositório por cima do `configuration.yaml` do HA. Mantém os ficheiros `automations.yaml`, `scripts.yaml` e `scenes.yaml` que o HA já tem.
3. Cria a pasta `packages/` e copia lá para dentro `homeassistant/packages/grupos.yaml`. **Faz isto só depois de renomeares as entidades** como manda o `docs/DEVICES.md`, porque os grupos usam esses nomes.
4. **Ferramentas de programador → YAML → Verificar configuração**, e depois **Reiniciar**.

## O que testar no fim da Fase 0

No Mac:

1. Abre `http://homeassistant.local:8123`.
2. Em **Visão geral**, liga e desliga o **Candeeiro Nuno** e confirma que a lâmpada responde logo.
3. Põe a **Persiana Cozinha** a 50 % no HA e confirma que ela se move.
4. Liga **Todas as luzes** (o grupo) e confirma que acendem todas as luzes.
5. Desliga a internet do router durante um minuto e confirma que continuas a controlar tudo pelo HA.
6. Diz-me quais dispositivos ficaram no **Zigbee2MQTT** e quais no **Tuya Local** (Wi-Fi).
