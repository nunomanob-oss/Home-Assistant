# Instalação — Fase 0: Home Assistant no Mac

Objetivo desta fase: ter o Home Assistant a correr numa máquina virtual no MacBook Pro M1, com todos os dispositivos Tuya visíveis e com nomes em português. Demora cerca de 1 hora.

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
- **Terminal & SSH** (para instalar o HACS no passo 6).

## 5. Ligar os dispositivos Tuya (rápido, pela nuvem)

Isto põe tudo a funcionar já, enquanto não temos o controlo local.

1. Na app SmartLife: **Eu → ⚙️ Definições → Conta e segurança → Código de utilizador**. Copia o código.
2. No HA: **Definições → Dispositivos e serviços → Adicionar integração → Tuya**. Cola o código de utilizador.
3. O HA mostra um QR code. Lê-o com a app SmartLife (botão de leitura no canto superior direito do Início) e confirma.
4. Devem aparecer o gateway, as 5 persianas, os interruptores e os 2 candeeiros.

## 6. Controlo local para os dispositivos Wi-Fi (Tuya Local)

Os dispositivos **Wi-Fi** passam a ser controlados diretamente na rede de casa, sem a nuvem.

1. **Instalar o HACS:** abre o Terminal (barra lateral) e corre:
   ```
   wget -O - https://get.hacs.xyz | bash -
   ```
   Reinicia o HA (**Definições → Sistema → Reiniciar**). Depois vai a **Adicionar integração → HACS** e segue o pedido de autorização do GitHub.
2. No HACS, procura e instala **Tuya Local** (de `make-all`). Reinicia o HA.
3. **Adicionar integração → Tuya Local** e escolhe a opção de configuração **assistida pela nuvem (Smart Life)**. Lês outra vez um QR com a app SmartLife, e o Tuya Local vai buscar as *local keys* sozinho.
4. Adiciona cada dispositivo **Wi-Fi** e escolhe o tipo sugerido.
5. Quando um dispositivo funcionar pelo Tuya Local, desativa a entidade equivalente da integração Tuya (nuvem) para não ficares com duplicados.

### Alternativa: conta de programador Tuya IoT (só se o passo 3 falhar)

1. Cria conta grátis em <https://iot.tuya.com> e cria um **Cloud Project** com o data center **Central Europe**.
2. Em **Devices → Link App Account**, lê o QR com a SmartLife.
3. Em **API Explorer → Device Management → Query device details**, copia o `local_key` de cada dispositivo.
4. No Tuya Local escolhe a configuração manual e cola o ID, o IP e a local key.

> As local keys mudam se voltares a emparelhar o dispositivo na SmartLife. Nesse caso, repete o passo 3.

## 7. Zigbee2MQTT (quando chegar o adaptador Zigbee)

Os dispositivos **Zigbee** saem do gateway Tuya e passam a falar com um adaptador Zigbee teu, 100 % local.

**Recomendação para quem usa VM: SLZB-06 (Ethernet/PoE)** em vez de uma pen USB. Liga-se pela rede, por isso não há passagem de USB para a VM e a mudança para o mini-PC é trivial. A pen SONOFF ZBDongle-E também funciona: no UTM, liga-a à VM pelo ícone de USB da barra de ferramentas.

1. **Suplementos → Loja → Mosquitto broker**: instala e arranca.
2. **Loja → ⋮ → Repositórios**: adiciona `https://github.com/zigbee2mqtt/hassio-zigbee2mqtt` e instala o **Zigbee2MQTT**.
3. Configuração do Zigbee2MQTT (separador Configuração, secção `serial`):
   - SLZB-06: `port: tcp://<IP-do-SLZB-06>:6638` e `adapter: ember` (firmware EFR32) ou `zstack` (firmware CC2652). Vê qual tens na página web do SLZB-06.
   - ZBDongle-E: `port: /dev/serial/by-id/<o-que-aparecer>` e `adapter: ember`.
4. Arranca o Zigbee2MQTT e abre-o pela barra lateral.
5. **Para cada dispositivo Zigbee**, um de cada vez:
   1. Na SmartLife, remove o dispositivo do gateway ("Remover dispositivo").
   2. No Zigbee2MQTT, carrega em **Permitir adesão (Permit join)**.
   3. Põe o dispositivo em modo de emparelhamento. Normalmente é manter o botão premido 5 a 10 segundos até a luz piscar depressa. Nas persianas costuma ser o botão de parar.
   4. Quando aparecer, dá-lhe o nome final (ver `docs/DEVICES.md`).
6. Quando já não houver nada no gateway Tuya, podes desligá-lo.

## 8. Token para a app (prepara já, usa-se na Fase 1)

1. No HA, clica no teu nome (canto inferior esquerdo) → **Segurança → Tokens de acesso de longa duração → Criar token**. Nome: `dashboard`.
2. Copia o token **e guarda-o no gestor de palavras-passe**. O HA só o mostra uma vez.
3. Nunca o ponhas no Git nem no código da app. Na Fase 1 fica no ficheiro `.env` do servidor, e quem o usa é o Caddy, não o browser.

## 9. Copiar a configuração deste repositório para o HA

1. Abre o **Studio Code Server**.
2. Copia `homeassistant/configuration.yaml` deste repositório por cima do `configuration.yaml` do HA. Mantém os ficheiros `automations.yaml`, `scripts.yaml` e `scenes.yaml` que o HA já tem.
3. Cria a pasta `packages/` e copia lá para dentro `homeassistant/packages/grupos.yaml`. **Faz isto só depois de renomeares as entidades** como manda o `docs/DEVICES.md`, porque os grupos usam esses nomes.
4. **Ferramentas de programador → YAML → Verificar configuração**, e depois **Reiniciar**.

## O que testar no fim da Fase 0

No Mac:

1. Abre `http://homeassistant.local:8123`.
2. Em **Visão geral**, liga e desliga o **Candeeiro Nuno** e confirma que muda também na SmartLife.
3. Põe a **Persiana Cozinha** a 50 % no HA e confirma que ela se move.
4. Liga **Todas as luzes** (o grupo) e confirma que acendem todas as luzes.
5. Diz-me quais dispositivos ficaram no **Tuya Local** (Wi-Fi) e quais ficaram na Tuya nuvem / Zigbee.
