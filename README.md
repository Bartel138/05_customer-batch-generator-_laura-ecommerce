# 05_customer-batch-generator-_laura-ecommerce
Description courte : Script Python d'ingénierie des données pour l'insertion massive de 50 000 clients (Faker français, secrets pour la sécurité, et INSERT IGNORE par paquets de 5 000).

# 👥 Laura E-Commerce - Massive Customer Batch Generator

## 🎯 Objectif du projet
Ce projet de Data Engineering résout la problématique de la génération et de l'insertion massive de **50 000 clients** en base de données de manière optimisée, sécurisée et performante.

## 🛠️ Fonctionnalités techniques & Bonnes pratiques
* **Traitement par paquets (*Batch Processing*)** : Regroupement des requêtes SQL par blocs de 5 000 lignes pour fluidifier l'écriture, réduire la charge et éviter de saturer le SGBD.
* **Génération de données réalistes** : Utilisation de la bibliothèque `Faker` (`fr_FR`) pour des noms, prénoms et e-mails conformes au marché français.
* **Sécurité cryptographique** : Utilisation du module natif `secrets` pour générer des hashs de mots de passe uniques et sécurisés.
* **Idempotence (`INSERT IGNORE`)** : Intégration d'une sécurité permettant de relancer le script sans bloquer sur les doublons existants.
* **Robustesse SQL** : Échappement automatique des apostrophes dans les noms (ex: `D''Artagnan`) pour prévenir les erreurs de syntaxe.

---

## 💻 Le Script Python (`generate_customers.py`)

```python
# ==========================================
# 1. IMPORTATION DES MODULES ET OUTILS
# ==========================================
import secrets
from faker import Faker
import datetime

# ==========================================
# 2. INITIALISATION DU GÉNÉRATEUR
# ==========================================
fake = Faker('fr_FR')
insert_customer = []

# ==========================================
# 3. PRÉPARATION DU FICHIER SQL (EN-TÊTE)
# ==========================================
insert_customer.append("USE laura_ecommerce_france;\n")
insert_customer.append("SET FOREIGN_KEY_CHECKS = 0;\n\n")
insert_customer.append("-- Generation de 50000 clients par paquets (Batch Insert)\n\n")

# ==========================================
# 4. LA BOUCLE DE GÉNÉRATION DES 50 000 CLIENTS
# ==========================================
customer_id = 1
total_clients = 50000
taille_paquet = 5000

while customer_id <= total_clients:
    insert_customer.append(
        "INSERT IGNORE INTO customer (customer_id, first_name, last_name, email, password_hash, created_at, last_connection, updated_at) VALUES\n"
    )
    
    paquet_lignes = []
    fin_paquet = min(customer_id + taille_paquet, total_clients + 1)
    
    for current_id in range(customer_id, fin_paquet):
        prenom = fake.first_name()
        nom = fake.last_name()
        
        email = f"{prenom.lower()}.{nom.lower()}{current_id}@gmail.com".replace(" ", "").replace("'", "")
        password = secrets.token_hex(4)
        
        created_at = fake.date_time_between(start_date="-16y", end_date="now")
        last_connection = fake.date_time_between(start_date="-16y", end_date="now")
        updated_at = fake.date_time_between(start_date="-16y", end_date="now")
        
        nom_securise = nom.replace("'", "''")
        
        ligne_valeur = (
            f"({current_id}, '{prenom}', '{nom_securise}', '{email}', '{password}', "
            f"'{created_at}', '{last_connection}', '{updated_at}')"
        )
        paquet_lignes.append(ligne_valeur)
        
    insert_customer.append(",\n".join(paquet_lignes) + ";\n\n")
    customer_id = fin_paquet

insert_customer.append("SET FOREIGN_KEY_CHECKS = 1;\n")

# ==========================================
# 5. ÉCRITURE DANS LE FICHIER SQL
# ==========================================
with open("insert_customers.sql", "w", encoding="utf-8") as f:
    f.writelines(insert_customer)

print(f"Succès ! Le fichier 'insert_customers.sql' contenant {total_clients} clients a été généré.")
