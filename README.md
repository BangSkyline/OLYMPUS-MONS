# Homelab Infra Entreprise-like : Proxmox + pfSense + K3s + Active Directory

Ce README documente la conception d'une infrastructure homelab simulant un environnement d'entreprise, en remplaçant une infra Docker standalone par un setup virtualisé sur Proxmox.

L'objectif principal : orchestrer des services conteneurisés avec **K3s** (monitoring, admin, apps enterprise comme GLPI, Planka, Zulip...), tout en intégrant une gateway **pfSense**, un domaine **Active Directory** (Windows Server + Windows 11 Pro), et une station d'administration.

## Aperçu de l'Architecture

![Olympus Banner](Infra-Pysique.png)

## Composants Principaux

- **Hyperviseur** : Proxmox VE (clustering possible pour HA future)
- **Gateway/Firewall** : pfSense VM
- **Administration** : VM Ubuntu Desktop (accès web/RDP/SSH)
- **Orchestrateur** : K3s cluster (1 server + 3 workers sur VMs Ubuntu Server)
- **Domaine** : Windows Server 2025 (Domain Controller) + Windows 11 Pro (client/test)
- **Services sur K3s** : Grafana, Prometheus, Technitium DNS, Nginx/Traefik Ingress, GLPI, Planka, Zulip, etc.

## Organisation Réseau Recommandée

### Bridges Proxmox
- `vmbr0` : WAN (NIC physique passthrough vers pfSense pour performances/isolation)
- `vmbr1` : LAN interne (VLAN aware, sans IP sur l'hôte Proxmox)

### Segmentation VLANs (802.1Q)
| VLAN | Subnet Exemple      | Usage                          | VMs associées                  |
|------|---------------------|--------------------------------|--------------------------------|
| 10   | 192.168.10.0/24     | Management                     | Ubuntu Desktop Admin           |
| 20   | 192.168.20.0/24     | K3s Cluster                    | K3s server + 3 workers         |
| 30   | 192.168.30.0/24     | Windows AD                     | WS2025 + W11 Pro               |
| 40   | 192.168.40.0/24     | Services/DMZ (optionnel)       | Futures VMs isolées            |

- pfSense : Interfaces WAN (vmbr0) + LAN (vmbr1 avec sous-interfaces VLAN)
- Routing inter-VLAN et firewall rules gérés par pfSense
- DHCP : Par VLAN sur pfSense (réservations pour IPs fixes)

## Suggestions et Améliorations

- **HA** : Embedded etcd sur K3s (avec les 3 workers), MetalLB pour LoadBalancer
- **Storage** : Longhorn sur K3s pour PV distribués
- **Sécurité** : VPN (WireGuard/OpenVPN) sur pfSense, pfBlockerNG, Network Policies sur K3s (Calico optionnel)
- **Monitoring** : Prometheus + Grafana sur K3s, exporters pour Proxmox/pfSense
- **Automation** : Ansible/Terraform pour provisionning, GitOps (ArgoCD) sur K3s
- **Backups** : Proxmox Backup Server, Velero sur K3s
- **Intégration AD** : Joindre VMs Linux au domaine (realmd + sssd), Kerberos pour auth centralisée

## Intégration Services Natifs K3s

- **CoreDNS** : Forward queries cluster.local vers pfSense, ou désactiver pour utiliser Technitium/AD DNS
- **Ingress** : Port forward 80/443 depuis pfSense, ou MetalLB pour IPs internes
- **Flannel CNI** : Overlay interne, pas de conflit avec pfSense (binder à interface VLAN 20)

## Ressources Allouées Suggerées (par VM)

- pfSense : 2-4 cœurs, 4GB RAM
- Ubuntu Desktop : 4 cœurs, 8GB RAM, GPU passthrough si besoin
- K3s Server : 4 cœurs, 8GB RAM
- K3s Workers : 2-4 cœurs, 4-6GB RAM chacun
- WS2025 : 4 cœurs, 8GB RAM
- W11 Pro : 4 cœurs, 8GB RAM

## Références
- Docs K3s : https://docs.k3s.io
- Proxmox + pfSense guides : Communautés Reddit /r/homelab et forums Proxmox
- Inspiration diagrammes : Divers homelabs open-source sur GitHub

Ce setup évolue facilement vers une infra plus pro (clustering Proxmox, Ceph storage, etc.).
