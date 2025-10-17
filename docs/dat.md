# Document d'Architecture Technique (DAT)

## 0 — Métadonnées
- Projet : Lab AD + Linux app (VulnerableLightApp)  
- Auteur : Elifsu JAFFRES  
- Branche repo : `docs`  
- Date : 2025-10-16  
- Emplacement des captures : `images/`  
- Emplacement gestionnaire de secrets : sur MacBook

---

## 1 — Résumé du périmètre
But : déployer un environnement de test vulnérable et réaliste composé de :
- 1 Windows Server = Contrôleur de domaine (AD + DNS + WinRM + SMB + BadBlood)
- 1 Windows Workstation = client joint au domaine (non déployé — tests réalisés depuis Mac)
- 1 Linux Server (Ubuntu) = application métier (VulnerableLightApp) + SSH

Livrables : preuves listées en §7, DAT au format PDF envoyé.

---

## 2 — Schéma réseau
- Voir image : `images/network-diagram.png`  
- Hyperviseur : UTM (VMs hébergées sur la machine hôte)  
- Mode réseau : NAT (réglage utilisé pour les mises à jour)

---

## 3 — Caractéristiques des machines
- Windows Server (AD DC)  
  - vCPU: 2, RAM: 4 GiB, Disk: 40 GiB, IP: 192.168.64.10  
  - Rôles : AD DS, DNS, SMB, WinRM

- Windows Client (théorie / non déployé)  
  - vCPU: 2, RAM: 4 GiB, Disk: 40 GiB  
  - Joint au domaine (si déployé)

- Ubuntu Server  
  - vCPU: 1, RAM: 2 GiB, Disk: 20 GiB, IP: 192.168.64.30  
  - Services : SSH (sudoers), dotnet / VulnerableLightApp (Kestrel)

---

## 4 — Plan d'adressage
- Réseau lab : 192.168.64.0/24  
- Passerelle (virtuel) : 192.168.64.1  
- DC : 192.168.64.10  
- Linux : 192.168.64.30

---

## 5 — Exemples de comptes créés
- labadmin (local/AD) — privilège admin — mot de passe de test : `LabAdmin123` (stocké localement)  
- labuser (local) — compte non‑admin

---

## 6 — Services déployés & état (preuves et commandes)

### DNS (AD)
- Commandes : `Get-Service DNS` / `Get-DnsServerZone`  
- Capture : `images/get-servicedns.png`

### WinRM
- Test : `Test-WSMan -ComputerName 192.168.64.10`  
- Capture test : `images/test-connexWinRm.png`

### SMB (DC)
- Vérif DC : `Get-SmbShare` → `images/smbshare.png`  
- Vérif montage Ubuntu : `images/smbubuntu.png`, `images/readonly.png`, `images/readwrite.png`

### Web (Ubuntu)
- Service systemd : `/etc/systemd/system/vla.service`  
- Commandes : `sudo systemctl status vla.service`, `journalctl -u vla.service`, `ss -ltnp`  
- Preuves service & écoute : `images/connexion-dotnet.png`, `images/dotnet-preuve.png`, `images/dot-vul.png`  
- Tests HTTP (réponse attendue) : `curl -I http://192.168.64.30:4000` → 401  
- Captures : `images/curl-vuln-macbook.png`, `images/web-req-win-vulapp.png`, `images/unauth-mac-vull.png`

### SSH (Ubuntu)
- Accessible pour membres du groupe `sudo`  
- Captures / preuves : `images/connexionsshubuntuviamac.png`, `images/sshserverubuntu.png`, `images/test-connexionubuntu.png`

### BadBlood
- Script exécuté sur le DC, sortie capturée : `images/scriptbadblood.png`

---

## 7 — Preuves / captures (fichiers présents dans `images/`)

- ISO / Hashs  
  - `images/isos-sha256.png` (si disponible)

- VM / hyperviseur screenshots  
  - `images/vullapplance.png`  
  - `images/domaincontroller-win.png`  
  - `images/installerwin10.png`

- Mises à jour  
  - `images/misajourwinserver.png`

- IP / config réseau  
  - `images/info-res-ubuntu.png`  
  - `images/info-res-ubuntu2.png`  
  - `images/ipconfig-win.png`

- SSH connect (Ubuntu)  
  - `images/connexionsshubuntuviamac.png`  
  - `images/test-connexionubuntu.png`

- WinRM / remote PowerShell  
  - `images/test-connexWinRm.png`  
  - `images/getadddomain.png`

- SMB shares & tests  
  - `images/smbshare.png`  
  - `images/smbubuntu.png`  
  - `images/readonly.png`  
  - `images/readwrite.png`

- VLA (service + HTTP tests)  
  - `images/connexion-dotnet.png`  
  - `images/dotnet-preuve.png`  
  - `images/dot-vul.png`  
  - `images/curl-vuln-macbook.png`  
  - `images/web-req-win-vulapp.png`  
  - `images/unauth-mac-vull.png`

- AD users / groups  
  - `images/aduser.png`  
  - `images/ad-group.png`  
  - `images/getadddomain.png`

- BadBlood execution  
  - `images/scriptbadblood.png`

- Divers / annexes  
  - `images/get-servicedns.png`  
  - `images/domaincontroller-win.png`  
  - `images/pinubuntutowin.png`

> Si un fichier n'existe pas exactement sous ce nom dans `images/`, adapte le nom dans le DAT ou renomme l’image dans le repo.

---

## 8 — Procédures / commandes utiles (Annexe)

### Vérif écoute VLA
```bash
sudo systemctl status vla.service -l
sudo journalctl -u vla.service -n 200 --no-pager
ss -ltnp | grep dotnet


