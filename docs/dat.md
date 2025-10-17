# Document d'Architecture Technique (DAT)

## 0 — Métadonnées
- Projet : Lab AD + Linux app (VulnerableLightApp)  
- Auteur : Elifsu JAFFRES  
- Branche repo : `docs`  
- Date : 2025-10-16  
- Emplacement des captures : `docs/images/` 
- Emplacement gestionnaire de secrets : sur MacBook

---

## 1 — Résumé du périmètre
But : déployer un environnement de test vulnérable et réaliste composé de :
- 1 Windows Server = Contrôleur de domaine (AD + DNS + WinRM + SMB + BadBlood)
- 1 Windows Workstation = client joint au domaine (non déployé — tests réalisés depuis Mac)
- 1 Linux Server (Ubuntu) = application métier (VulnerableLightApp) + SSH

Livrables : preuves  listées en §7, DAT au format PDF enovoyé.

---

## 2 — Schéma réseau
- Voir image : `docs/images/network-diagram.png`   
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

(adapte ces valeurs si tu veux indiquer la configuration exacte de tes VMs)

---

## 4 — Plan d'adressage
- Réseau lab : 192.168.64.0/24  
- Passerelle (virtuel) : 192.168.64.1  
- DC : 192.168.64.10  
- Linux : 192.168.64.30

---

## 5 — Examples de Comptes créés
- labadmin (local/AD) — privilège admin — mot de passe de test : `LabAdmin123` (stocké localement) 
- labuser (local) — compte non‑admin

---

## 6 — Services déployés & état (preuves et commandes)
- DNS (AD) : installé sur DC  
  - Commandes : `Get-Service DNS` / `Get-DnsServerZone`  
  - Capture : `docs/images/get-servicedns.png`
- WinRM : activé sur DC, restreint (préconisation)  
  - Test : `Test-WSMan -ComputerName 192.168.64.10`  
  - Capture test : `docs/images/test-connexWinRm.png`
- SMB (DC) : partages `Readonly` & `Write` (Everyone)  
  - Vérif DC : `Get-SmbShare` → `docs/images/smbshare.png`  
  - Vérif montage Ubuntu : `docs/images/smbubuntu.png`, `docs/images/readonly.png`, `docs/images/readwrite.png`
- Web (Ubuntu) :
  - Service systemd : `/etc/systemd/system/vla.service`  
  - Commandes : `sudo systemctl status vla.service`, `journalctl -u vla.service`, `ss -ltnp`  
  - Preuves service & écoute : `docs/images/connexion-dotnet.png`, `docs/images/dotnet-preuve.png`, `docs/images/dot-vul.png`
  - Tests HTTP (réponse attendue) : `curl -I http://192.168.64.30:4000` → 401  
  - Captures : `docs/images/curl-vuln-macbook.png`, `docs/images/web-req-win-vulapp.png`, `docs/images/unauth-mac-vull.png`
- SSH (Ubuntu) :
  - Accessible pour membres du groupe `sudo`  
  - Captures / preuves : `docs/images/connexionsshubuntuviamac.png`, `docs/images/sshserverubuntu.png`, `docs/images/test-connexionubuntu.png`
- BadBlood :
  - Script exécuté sur le DC, sortie capturée : `docs/images/scriptbadblood.png`

---

## 7 — Preuves / captures (fichiers présents dans `docs/images/`)
Place ou vérifie ces captures dans `docs/images/` ; ci‑dessous les noms (tels que fournis) et leur mapping vers la preuve requise :

- ISO / Hashs
  - (ajoute capture sha256) → `docs/images/isos-sha256.png` (si tu as screenshot)
- VM / hyperviseur screenshots
  - `docs/images/vullapplance.png` (capture appliance / VM)
  - `docs/images/domaincontroller-win.png`
  - `docs/images/installerwin10.png`
- Mises à jour
  - `docs/images/misajourwinserver.png`
- IP / config réseau
  - `docs/images/info-res-ubuntu.png`
  - `docs/images/info-res-ubuntu2.png`
  - `docs/images/ipconfig-win.png`
- SSH connect (Ubuntu)
  - `docs/images/connexionsshubuntuviamac.png`
  - `docs/images/test-connexionubuntu.png`
- WinRM / remote PowerShell
  - `docs/images/test-connexWinRm.png`
  - `docs/images/getadddomain.png` (si lié)
- SMB shares & tests
  - `docs/images/smbshare.png`
  - `docs/images/smbubuntu.png`
  - `docs/images/readonly.png`
  - `docs/images/readwrite.png`
- VLA (service + HTTP tests)
  - `docs/images/connexion-dotnet.png`
  - `docs/images/dotnet-preuve.png`
  - `docs/images/dot-vul.png`
  - `docs/images/curl-vuln-macbook.png`
  - `docs/images/web-req-win-vulapp.png`
  - `docs/images/unauth-mac-vull.png`
- AD users / groups
  - `docs/images/aduser.png`
  - `docs/images/ad-group.png`
  - `docs/images/getadddomain.png` (liste/domain info)
- BadBlood execution
  - `docs/images/scriptbadblood.png`
- Divers / annexes
  - `docs/images/get-servicedns.png`
  - `docs/images/domaincontroller-win.png`
  - `docs/images/pinubuntutowin.png`

> Si un fichier n'existe pas exactement sous ce nom dans `docs/images/`, adapte le nom dans le DAT ou renomme l’image dans le repo. Les noms ci‑dessus correspondent aux captures que tu as listées.

---

## 8 — Procédures / commandes utiles (Annexe)
- Vérif écoute VLA :
```bash
sudo systemctl status vla.service -l
sudo journalctl -u vla.service -n 200 --no-pager
ss -ltnp | grep dotnet
```
- Test HTTP :
```bash
curl -I http://127.0.0.1:4000
curl -I http://192.168.64.30:4000
```
- SMB tests (Ubuntu) :
```bash
sudo apt install -y smbclient cifs-utils
smbclient -L //192.168.64.10 -N
sudo mount -t cifs //192.168.64.10/Readonly /mnt/readonly -o guest,ro
sudo mount -t cifs //192.168.64.10/Write /mnt/write -o guest
touch /mnt/write/testfile.txt
```
- WinRM (PowerShell) :
```powershell
Test-WSMan -ComputerName 192.168.64.10
Enter-PSSession -ComputerName 192.168.64.10 -Credential (Get-Credential)
whoami
```
- Lister utilisateurs AD (DC) :
```powershell
Import-Module ActiveDirectory
Get-ADUser -Filter * | Select SamAccountName | Out-File C:\Temp\ad-users-list.txt
Get-ADUser -Filter * | Measure-Object
```

---

## 9 — Timeline (Gantt / Kanban)
- Phase 1 (J1) : préparation ISOs + vérifications hash + création VMs  
- Phase 2 (J2) : installation OS + mises à jour  
- Phase 3 (J3) : configuration AD, DNS, WinRM, SMB  
- Phase 4 (J4) : déploiement VLA + tests réseau  
- Phase 5 (J5) : exécution BadBlood + captures + rédaction DAT + export PDF  
(joindre un screenshot Kanban/Gantt dans `docs/images/`)

---

## 10 — Sécurisation après lab (recommandations)
- Remplacer mots de passe par secrets forts et stocker dans KeePass.  
- Restreindre ACL SMB.  
- Restreindre WinRM par GPO/SDDL à Domain Admins.  
- Isoler le lab (NAT/Host‑only) et supprimer VMs si doute sur intégrité.  
- Auditer et patcher OS / dépendances .NET.

---

## 11 — Annexes / logs / checksums
- Place ici les sorties de `sha256sum`, `ip addr`, `ipconfig /all`, `Get-SmbShare`, `journalctl` et autres logs (fichiers texte ou screenshots dans `docs/images/`).  
- Exemple de fichier texte ajouté au repo : `docs/outputs/isos-sha256.txt`, `docs/outputs/vla-journal.txt`, `docs/outputs/ad-users-list.txt`.


