# ADR-001 : Stratégie de migration Cloud - Approche Lift & Shift

## Méta-données

| Attribut     | Valeur                                                                                                 |
| ------------ | ------------------------------------------------------------------------------------------------------ |
| ID           | ADR-001                                                                                                |
| Date         | 2024-12-10                                                                                             |
| Statut       | Proposé                                                                                                |
| Portée       | Application                                                                                            |
| Participants | Sébastien Marin, Architecte Logiciel<br>Yohann Streibel, DevOps<br>Arnaud Gaches, Architecte Technique |

## Contexte

### Situation actuelle
L'application existante est une solution Java qui assure trois fonctions principales :
- Lecture de fichiers depuis un système de fichiers (SFTP)
- Transformation des données contenues dans ces fichiers
- Transmission des données transformées vers une file de message Kafka

Une migration Cloud de type "lift and shift" a été initiée. L'apparition récente d'un nouveau service managé soulève des questions sur la pertinence de l'approche choisie.

### Contraintes
#### Contraintes métier
- Décommissionnement prévu dans 1,5 an
- Budget limité à réserver prioritairement pour le redéveloppement futur
- Planning serré avec une migration à terminer rapidement

#### Contraintes techniques
- Équipe réduite avec des compétences principalement orientées Java
- Nécessité de maintenir la compatibilité SFTP
- Infrastructure existante à migrer sans interruption de service

### Exigences
- **Normes** : Conformité avec les standards de l'entreprise pour les migrations Cloud
- **Sécurité** : Maintien des niveaux de sécurité actuels, notamment pour l'accès SFTP
- **Métier** : Garantie de la continuité de service pendant la migration
- **Technique** : Minimisation des modifications du code existant
- **Standards** : Respect des bonnes pratiques Cloud lorsque possible

### Hypothèses
- Le nouveau service managé restera disponible et supporté pendant la durée de vie de l'application
- Les coûts d'infrastructure Cloud resteront stables
- L'équipe pourra maintenir l'application jusqu'à son décommissionnement

## Problématiques
1. Faut-il modifier l'approche de migration initiale suite à l'apparition d'un nouveau service managé ?
2. Comment optimiser l'investissement compte tenu de la durée de vie limitée ?
3. Quel est le meilleur compromis entre modernisation et pragmatisme ?

## Scénarios envisagés

### Vue d'ensemble des scénarios

| Scénario            | Description                                | Avantages                                                                     | Inconvénients                                                                        | Complexité | Coût |
| ------------------- | ------------------------------------------ | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | ---------- | ---- |
| **Lift & Shift VM** | Migration directe vers des VMs             | • Simplicité d'exécution<br>• Risques minimaux<br>• Rapidité de mise en œuvre | • Maintenance plus lourde<br>• Coûts d'infrastructure plus élevés                    | Faible     | €    |
| **Migration GKE**   | Conteneurisation et déploiement sur GKE    | • Meilleure scalabilité<br>• Standardisation des déploiements                 | • Effort de conteneurisation<br>• Pas de module SFTP standard<br>• Complexité accrue | Élevé      | €€€  |
| **ETL Managé**      | Remplacement par solution ETL Cloud native | • Services managés optimisés<br>• Réduction des coûts opérationnels           | • Développement important<br>• Risques de régression élevés                          | Élevé      | €€€  |
| **Hybride VM/GKE**  | SFTP sur VM, traitement sur GKE            | • Modernisation progressive<br>• Compromis flexibilité/complexité             | • Deux environnements à gérer<br>• Complexité de coordination                        | Moyen      | €€   |

### Scénario 1 : Lift & Shift VM
#### Description
Migration directe de l'application existante vers des machines virtuelles dans le Cloud, en conservant l'architecture actuelle.

#### Analyse
- **Avantages**
  - Approche la plus simple et la plus rapide
  - Risques techniques minimaux
  - Conservation des processus existants
  - Facilité de rollback
  - Pas de montée en compétence nécessaire
- **Inconvénients**
  - Maintenance opérationnelle plus importante
  - Coûts d'infrastructure potentiellement plus élevés
  - Non-utilisation des services managés
  - Dette technique acceptée

### Scénario 2 : Migration GKE
#### Description
Conteneurisation de l'application et déploiement sur Google Kubernetes Engine.

#### Analyse
- **Avantages**
  - Meilleure scalabilité
  - Déploiements standardisés
  - Monitoring amélioré
  - Portabilité accrue
- **Inconvénients**
  - Absence de module SFTP standard pour Kubernetes
  - Effort significatif de conteneurisation
  - Complexité accrue pour une durée de vie courte
  - Nécessité de formation de l'équipe
  - Risques techniques plus élevés

### Scénario 3 : ETL Managé
#### Description
Refonte complète de la solution en utilisant des services ETL managés du Cloud provider.

#### Analyse
- **Avantages**
  - Utilisation optimale des services Cloud
  - Réduction des coûts opérationnels
  - Maintenance minimale
  - Meilleure scalabilité
- **Inconvénients**
  - Effort de développement majeur
  - Risques élevés de régression
  - Formation importante nécessaire
  - Coût initial élevé
  - Complexité de migration des données

### Scénario 4 : Hybride VM/GKE
#### Description
Approche hybride avec maintien du SFTP sur VM et migration du traitement Java vers GKE.

#### Analyse
- **Avantages**
  - Compromis entre modernisation et pragmatisme
  - Conservation des fonctionnalités SFTP existantes
  - Modernisation progressive possible
- **Inconvénients**
  - Complexité de gestion de deux environnements
  - Coordination nécessaire entre composants
  - Coûts de maintenance plus élevés
  - Complexité opérationnelle accrue

## Décision

### Choix retenu
Scénario 1 : Lift & Shift VM

### Justification
Le choix du Lift & Shift est motivé par plusieurs facteurs clés :
1. La durée de vie limitée de l'application (1,5 an) ne justifie pas un investissement majeur
2. L'approche minimise les risques techniques et les efforts de migration
3. Les ressources limitées de l'équipe sont mieux utilisées pour la maintenance que pour une transformation
4. La simplicité de l'approche permet de respecter les contraintes de temps
5. Le surcoût opérationnel est acceptable compte tenu de la durée de vie limitée

### Scénarios non retenus
- **Migration GKE** : Complexité et efforts trop importants pour la durée de vie restante
- **ETL Managé** : Investissement et risques disproportionnés par rapport aux bénéfices
- **Hybride VM/GKE** : Complexité accrue sans bénéfices justifiant l'investissement

## Conséquences et impacts

### Impacts
- **Impacts positifs**
  - Minimisation des risques de migration
  - Conservation des compétences existantes
  - Rapidité de mise en œuvre
  - Facilité de maintenance pour l'équipe actuelle
  - Coûts de migration minimaux
- **Impacts négatifs**
  - Dette technique acceptée jusqu'au décommissionnement
  - Coûts d'infrastructure légèrement plus élevés
  - Non-utilisation des services managés disponibles
  - Nécessité d'actions de maintenance plus fréquentes
