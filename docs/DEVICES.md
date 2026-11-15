# Dispositivos e nomes

A app, a Siri e os grupos usam os **IDs de entidade** desta tabela. Depois de adicionares os dispositivos no HA (INSTALL.md, passos 5 e 6), renomeia cada entidade para o ID indicado. Para isso vai a **Definições → Dispositivos e serviços → Entidades**, clica na entidade, abre ⚙️ e muda o **Nome** e o **ID da entidade**.

O nome é o que a Siri e a app mostram, por isso deve soar natural em português.

## Interruptores de parede → luzes

Os interruptores aparecem no HA como `switch`. Para que a app, os grupos e a Siri os tratem como luzes, converte-os:

**Definições → Dispositivos e serviços → Ajudantes → Criar ajudante → Alterar o tipo de dispositivo de um interruptor** → escolhe o `switch` → tipo **Luz**.

Usa a luz nova nos nomes da tabela e esconde o `switch` original (⚙️ → Visível: desligado).

A **luz de fundo** (backlight) dos interruptores fica como `switch` e escondida. Não interessa à app.

## Tabela de entidades

| Divisão | Dispositivo na SmartLife | ID da entidade no HA | Nome (Siri e app) |
| --- | --- | --- | --- |
| Quarto 1 | Candeeiro Nuno | `light.candeeiro_nuno` | Candeeiro Nuno |
| Quarto 1 | Candeeiro Marisa | `light.candeeiro_marisa` | Candeeiro Marisa |
| Quarto 1 | Persiana Quarto 1 | `cover.persiana_quarto_1` | Persiana do Quarto 1 |
| Quarto 2 | Luz Quarto 2 · Switch 1 | `light.quarto_2_tecla_1` | Luz do Quarto 2 |
| Quarto 2 | Luz Quarto 2 · Switch 2 | `light.quarto_2_tecla_2` | Luz 2 do Quarto 2 |
| Quarto 2 | Persiana quarto 2 | `cover.persiana_quarto_2` | Persiana do Quarto 2 |
| Quarto 3 | Quarto 3 (interruptor) | `light.quarto_3` | Luz do Quarto 3 |
| Quarto 3 | Persiana quarto 3 | `cover.persiana_quarto_3` | Persiana do Quarto 3 |
| Corredor | Corredor 1 | `light.corredor` | Luz do Corredor |
| Corredor | Corredor 2 | (escondido) | — |
| WC | Persiana WC | `cover.persiana_wc` | Persiana do WC |
| Cozinha | Persiana Cozinha | `cover.persiana_cozinha` | Persiana da Cozinha |

**Corredor 1 e Corredor 2** controlam a mesma luz (são dois interruptores de escada). Convertemos só o Corredor 1 em luz e escondemos o Corredor 2. Assim a Siri e a app veem uma única "Luz do Corredor". Se ao desligar pelo Corredor 1 o estado ficar errado, avisa-me: nesse caso juntamos os dois num grupo.

**Quarto 2 tem duas teclas.** Se acenderem coisas diferentes (por exemplo teto e candeeiro), muda os nomes da tabela para algo como "Teto do Quarto 2" e "Candeeiro do Quarto 2".

## Para mais tarde

Quando instalares luzes na sala, WC, cozinha e lavandaria, ou sensores de água, porta e temperatura:

1. Adiciona o dispositivo no HA e dá-lhe um ID seguindo o padrão `<tipo>.<o_que_e>_<divisao>` (por exemplo `binary_sensor.agua_cozinha`, `sensor.temperatura_sala`).
2. Acrescenta uma linha a esta tabela.
3. Acrescenta-o a `config/house.json`, na divisão certa. A app passa a mostrá-lo sem mexer em código.
4. Se for uma luz ou uma persiana, acrescenta-o também ao grupo em `homeassistant/packages/grupos.yaml`.

## Videoporteiro EZVIZ CP5

Entra na Fase 5. Por agora basta instalá-lo e configurá-lo na app EZVIZ.
