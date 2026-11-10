# Comandes del servidor del TFG

Documentació de les comandes utilitzades per configurar el servidor del projecte.

> En preparació. Només s'hi afegiran comandes que s'hagin confirmat com a executades correctament. Revisa-les abans de reutilitzar-les: poden dependre de la configuració del servidor.

## Contingut

- [Comandes](./COMANDES.md): configuració inicial de Debian, zRAM, xarxa, seguretat, K3s, K9s, namespaces, emmagatzematge, diagnòstic DNS i desplegament d'AdGuard Home
- Manifests Kubernetes a [`manifests/`](./manifests/)
- Manifest de Jellyfin: [`manifests/jellyfin-deployment.yaml`](./manifests/jellyfin-deployment.yaml)

## Seguretat

No s'hi han d'incloure contrasenyes, tokens, claus, adreces IP privades ni altres dades identificatives. Les ordres que puguin esborrar dades s'identificaran clarament.

## Estat

La configuració inicial de Debian, zRAM, xarxa, seguretat, K3s, K9s, namespaces, carpetes
persistents, diagnòstic del port DNS, AdGuard Home i Jellyfin ja està documentada. Els manifests
disponibles són a [`manifests/`](./manifests/). AdGuard Home està configurat per servir la
interfície web al port `30080`.
