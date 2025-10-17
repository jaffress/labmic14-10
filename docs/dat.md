# Document d'Architecture Technique (DAT)

## 0 — Métadonnées
- **Projet** : Lab AD + Linux app (VulnerableLightApp)  
- **Auteur** : Elifsu JAFFRES  
- **Branche repo** : `main`  
- **Date** : 2025-10-16  
- **Emplacement des captures** : `images/`  
- **Emplacement gestionnaire de secrets** : sur MacBook

---

## 1 — Résumé du périmètre

**But** : déployer un environnement de test vulnérable et réaliste composé de :
- 1 Windows Server = Contrôleur de domaine (AD + DNS + WinRM + SMB + BadBlood)
- 1 Windows Workstation = client joint au domaine (non déployé — tests réalisés depuis Mac)
- 1 Linux Server (Ubuntu) = application métier (VulnerableLightApp) + SSH

**Livrables** : preuves listées en §7, DAT au format PDF envoyé.
Voici le diagramme de Gantt lié au planning du projet :

![Diagramme de Gantt](images/diagram.png)
---

## 2 — Schéma réseau

- **Hyperviseur** : UTM (VMs hébergées sur la machine hôte)  
- **Mode réseau** : NAT (réglage utilisé pour les mises à jour)

![Schéma réseau](images/network-diagram.png)

---

## 3 — Caractéristiques des machines

### Windows Server (AD DC)
- **vCPU** : 2
- **RAM** : 4 GiB
- **Disk** : 40 GiB
- **IP** : 192.168.64.10
- **Rôles** : AD DS, DNS, SMB, WinRM

![Windows Server - Domain Controller](images/domaincontroller-win.png)

### Windows Client (théorie / non déployé)
- **vCPU** : 2
- **RAM** : 4 GiB
- **Disk** : 40 GiB

![Windows 10 Installer](images/installerwin10.png)

### Ubuntu Server
- **vCPU** : 1
- **RAM** : 2 GiB
- **Disk** : 20 GiB
- **IP** : 192.168.64.30
- **Services** : SSH (sudoers), dotnet / VulnerableLightApp (Kestrel)

![Ubuntu VulnerableLightApp](images/vullapplance.png)

---

## 4 — Plan d'adressage

- **Réseau lab** : 192.168.64.0/24  
- **Passerelle (virtuel)** : 192.168.64.1  
- **DC** : 192.168.64.10  
- **Linux** : 192.168.64.30

### Configuration IP Windows Server

![Configuration IP Windows](images/ipconfig-win.png)

### Configuration IP Ubuntu

![Configuration réseau Ubuntu 1](images/info-res-ubuntu.png)

![Configuration réseau Ubuntu 2](images/info-res-ubuntu2.png)

![Ping Ubuntu vers Windows](images/pinubuntutowin.png)

---

## 5 — Exemples de comptes créés

- **labadmin** (local/AD) — privilège admin — mot de passe de test : `LabAdmin123` (stocké localement)  
- **labuser** (local) — compte non‑admin

---

## 6 — Services déployés & état (preuves et commandes)

### DNS (AD)

**Commandes** : `Get-Service DNS` / `Get-DnsServerZone`

![Service DNS](images/get-servicedns.png)

---

### WinRM

**Test** : `Test-WSMan -ComputerName 192.168.64.10`

![Test connexion WinRM](images/test-connexWinRm.png)

![Get-ADDomain via WinRM](images/getadddomain.png)

---

### SMB (DC)

**Vérif DC** : `Get-SmbShare`

![Partages SMB](images/smbshare.png)

**Vérif montage Ubuntu** :

![SMB monté sur Ubuntu](images/smbubuntu.png)

![Test lecture seule](images/readonly.png)

![Test lecture/écriture](images/readwrite.png)

---

### Web (Ubuntu)

**Service systemd** : `/etc/systemd/system/vla.service`

**Commandes** : `sudo systemctl status vla.service`, `journalctl -u vla.service`, `ss -ltnp`

**Preuves service & écoute** :

![Connexion dotnet](images/connexion-dotnet.png)

![Preuve service dotnet](images/dotnet-preuve.png)

![Application vulnérable active](images/dot-vul.png)

**Tests HTTP** :

![Curl depuis MacBook](images/curl-vuln-macbook.png)

![Requête web depuis Windows](images/web-req-win-vulapp.png)

![Accès non authentifié depuis Mac](images/unauth-mac-vull.png)

---

### SSH (Ubuntu)

**Captures / preuves** :

![Connexion SSH Ubuntu depuis Mac](images/connexionsshubuntuviamac.png)

![Serveur SSH Ubuntu](images/sshserverubuntu.png)

![Test connexion Ubuntu](images/test-connexionubuntu.png)

---

### BadBlood

**Script exécuté sur le DC**

![Exécution script BadBlood](images/scriptbadblood.png)

---

## 7 — Preuves / captures

### ISO / Hashs

![Hashs SHA256 des ISOs](images/isos-sha256.png)

### VM / hyperviseur screenshots

![Ubuntu VulnerableLightApp](images/vullapplance.png)

![Windows Server - Domain Controller](images/domaincontroller-win.png)

![Windows 10 Installer](images/installerwin10.png)

### Mises à jour

![Mises à jour Windows Server](images/misajourwinserver.png)

### IP / config réseau

![Configuration réseau Ubuntu 1](images/info-res-ubuntu.png)

![Configuration réseau Ubuntu 2](images/info-res-ubuntu2.png)

![Configuration IP Windows](images/ipconfig-win.png)

### SSH connect (Ubuntu)

![Connexion SSH Ubuntu depuis Mac](images/connexionsshubuntuviamac.png)

![Test connexion Ubuntu](images/test-connexionubuntu.png)

### WinRM / remote PowerShell

![Test connexion WinRM](images/test-connexWinRm.png)

![Get-ADDomain](images/getadddomain.png)

### SMB shares & tests

![Partages SMB](images/smbshare.png)

![SMB monté sur Ubuntu](images/smbubuntu.png)

![Test lecture seule](images/readonly.png)

![Test lecture/écriture](images/readwrite.png)

### VLA (service + HTTP tests)

![Connexion dotnet](images/connexion-dotnet.png)

![Preuve service dotnet](images/dotnet-preuve.png)

![Application vulnérable active](images/dot-vul.png)

![Curl depuis MacBook](images/curl-vuln-macbook.png)

![Requête web depuis Windows](images/web-req-win-vulapp.png)

![Accès non authentifié depuis Mac](images/unauth-mac-vull.png)

### AD users / groups

![Utilisateurs AD](images/aduser.png)

![Groupes AD](images/ad-group.png)

![Get-ADDomain](images/getadddomain.png)

### BadBlood execution

![Exécution script BadBlood](images/scriptbadblood.png)

### Divers / annexes

![Service DNS](images/get-servicedns.png)

![Windows Server - Domain Controller](images/domaincontroller-win.png)

![Ping Ubuntu vers Windows](images/pinubuntotowin.png)

---

## 8 — Procédures / commandes utiles

### Vérif écoute VLA
```bash
sudo systemctl status vla.service -l
sudo journalctl -u vla.service -n 200 --no-pager
ss -ltnp | grep dotnet
```

### Commandes Active Directory
```powershell
Get-ADDomain
Get-ADUser -Filter *
Get-ADGroup -Filter *
Get-Service DNS
Get-DnsServerZone
```

### Commandes SMB
```powershell
Get-SmbShare
```
```bash
# Montage depuis Ubuntu
sudo mount -t cifs //192.168.64.10/partage /mnt/smb -o username=labadmin
```

### Test WinRM
```powershell
Test-WSMan -ComputerName 192.168.64.10
Enter-PSSession -ComputerName 192.168.64.10 -Credential (Get-Credential)
```
