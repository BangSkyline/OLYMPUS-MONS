# Homelab : Proxmox + pfSense + Swarm + Active Directory

Ce README documente la migration Standalone vers Swarm d'une infrastructure homelab simulant un environnement d'entreprise, en remplaçant une infra Docker par un setup virtualisé sur Proxmox.

L'objectif principal : orchestrer des services conteneurisés avec **Swarm** (apps métier comme GLPI, Planka...), tout en intégrant une gateway **pfSense**, un domaine **Active Directory** (Windows Server + Windows 11 Pro), et une station d'administration.

## Aperçu de l'Architecture

![Olympus Banner](Infra-Physique.png)

## Composants Principaux

- **Hyperviseur** : Proxmox VE (clustering possible pour HA future)
- **Gateway/Firewall** : pfSense VM
- **Administration** : VM Ubuntu Desktop (accès web/RDP/SSH)
- **Orchestrateur** : Cluster Swarm (1 server + 3 workers sur VMs Ubuntu Server)
- **Domaine** : Windows Server 2025 (Domain Controller) + Windows 11 Pro (client/test)
- **Services** : Grafana, Prometheus, Technitium DNS, Traefik Ingress, GLPI, Planka, Mattermost, etc.

## Organisation Réseau

### Bridges Proxmox
- `vmbr0` : WAN (NIC physique passthrough vers pfSense pour performances/isolation)
- `vmbr1` : LAN interne (VLAN aware, sans IP sur l'hôte Proxmox)

### Segmentation VLANs (802.1Q)
| VLAN | Subnets             | Usage                          | VMs associées                  |
|------|---------------------|--------------------------------|--------------------------------|
| 10   | 192.168.10.0/24     | Management                     | Ubuntu Desktop Admin           |
| 20   | 192.168.20.0/24     | Cluster                        | Swarm manager + 3 workers      |
| 30   | 192.168.30.0/24     | Windows AD                     | WS2025 + W11 Pro               |
| 40   | 192.168.40.0/24     | Services/DMZ (optionnel)       | Futures VMs isolées            |

- pfSense : Interfaces WAN (vmbr0) + LAN (vmbr1 avec sous-interfaces VLAN)
- Routing inter-VLAN et firewall rules gérés par pfSense
- DHCP : Par VLAN sur pfSense (réservations pour IPs fixes)

## Suggestions et Améliorations

- **HA** : Embedded etcd sur Swarm (avec les 3 workers), MetalLB pour LoadBalancer
- **Sécurité** : VPN Tailscale (WireGuard), pfBlockerNG, Network Policies
- **Automation** : Ansible + Terraform pour provisionning, Jenkins
- **Backups** : Proxmox Backup Server
- **Intégration AD** : Joindre VMs Linux au domaine (realmd + sssd), Kerberos pour auth centralisée

## Ressources Allouées Suggerées (par VM)

- pfSense : 1 cœurs, 1GB RAM
- Ubuntu Desktop : 4 cœurs, 8GB RAM
- K3s Server : 4 cœurs, 8GB RAM
- K3s Workers : 2-4 cœurs, 4GB RAM chacun
- WS2025 : 4 cœurs, 8GB RAM
- W11 Pro : 4 cœurs, 8GB RAM

## Références
- Proxmox + pfSense guides : Communautés Reddit /r/homelab et forums Proxmox
- Diagrammes : FossFlow
Ce setup évolue facilement vers une infra plus pro (clustering Proxmox, Ceph storage, etc.).
