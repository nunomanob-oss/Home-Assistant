# Casa inteligente

Dashboard próprio para a casa, a substituir a app SmartLife. O Home Assistant (HA) é o hub e fala com os dispositivos Tuya, de preferência localmente. Por cima corre uma PWA com a planta 3D isométrica da casa, que se instala no iPad, iPhone, Mac, Android e Windows. A Siri funciona através do HomeKit Bridge.

## Estrutura

```
/homeassistant      configuração do HA (configuration.yaml, packages/)
/config/house.json  divisões, janelas, dispositivos e posição de cada um na planta
/docs               INSTALL.md, DEVICES.md (e, nas próximas fases, VOICE.md e KIOSK.md)
/web                app React + Vite + TypeScript (a partir da Fase 1)
/deploy             Caddy com HTTPS e proxy do token (a partir da Fase 1)
```

## Fases

| Fase | O que fica a funcionar | Estado |
| --- | --- | --- |
| 0 | Home Assistant numa VM no Mac, dispositivos Tuya no HA, nomes em PT | em curso |
| 1 | App PWA ligada ao HA em tempo real, lista de luzes, HTTPS, modo demo | — |
| 2 | Planta 3D isométrica com as divisões a acender | — |
| 3 | Persianas e cenas | — |
| 4 | Voz: Siri (HomeKit Bridge), microfone na app, Atalhos | — |
| 5 | Videoporteiro EZVIZ CP5, histórico e alertas | — |
| 6 | Modo kiosk no iPad e polimento | — |

## Começar

Segue o [docs/INSTALL.md](docs/INSTALL.md) e depois o [docs/DEVICES.md](docs/DEVICES.md).

## Regras do repositório

- Nada de tokens, palavras-passe ou local keys no Git: ficam em `.env` ou em `secrets.yaml` (ambos no `.gitignore`).
- A morada da casa não entra no repositório. A localização do HA é definida na interface do HA e não no YAML.
