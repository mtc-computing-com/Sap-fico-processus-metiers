# ⚙️ Processus Métiers SAP — P2P · O2C · R2R · D2P · H2R

![SAP](https://img.shields.io/badge/SAP-S%2F4HANA-0FAAFF?style=flat-square&logo=sap&logoColor=white)
![Version](https://img.shields.io/badge/Version-Expert%20Key%20User-blue?style=flat-square)
![Auteur](https://img.shields.io/badge/Auteur-Moez%20L'Agha-green?style=flat-square)
![MTC Computing](https://img.shields.io/badge/MTC%20Computing-Consultant%20SAP%20FI%2FCO-orange?style=flat-square)

> **Tables · Mécanismes · Contrôles · KPIs**
>
> Référentiel opérationnel des 5 processus métiers SAP couvrant les flux financiers de bout en bout — de la commande d'achat au bilan, du recrutement au virement de salaire.
> Conçu pour les consultants FICO et Key Users S/4HANA.

---

## 📋 Table des matières

1. [Synthèse des 5 processus](#synthèse-des-5-processus)
2. [P2P — Procure-to-Pay](#1-p2p--procure-to-pay--dépenser)
3. [O2C — Order-to-Cash](#2-o2c--order-to-cash--vendre-et-encaisser)
4. [R2R — Record-to-Report](#3-r2r--record-to-report--clôture-et-bilan)
5. [D2P — Design-to-Produce](#4-d2p--design-to-produce--fabriquer)
6. [H2R — Hire-to-Retire](#5-h2r--hire-to-retire--gérer-lhumain)

---

## Synthèse des 5 processus

| Acronyme | Traduction réelle | Flux financier | KPI juge de paix | Compte / Table sentinelle |
|---|---|---|---|---|
| **P2P** | Procure-to-Pay | Sortie d'argent | Solde 408100 | EKKO/EKPO · RBKP/RSEG · BKPF/BSEG |
| **O2C** | Order-to-Cash | Entrée d'argent | DSO (jours) | VBAK/VBAP · VBRK/VBRP · BSID |
| **R2R** | Record-to-Report | Clôture / Bilan | Fast Close (jours) | ACDOCA · BKPF/BSEG · T001B (OB52) |
| **D2P** | Design-to-Produce | Fabrication | Écart coût de revient | MAST · AUFK · COSS/COSP |
| **H2R** | Hire-to-Retire | Gestion humain | Justesse provision CP | PA0001 · RT · 641xxx/431xxx |

---

## 1. P2P — Procure-to-Pay : Dépenser

Périmètre : tout ce qui concerne la sortie d'argent pour obtenir un bien ou un service — de l'expression du besoin interne jusqu'au décaissement bancaire.

### 1.1 Mécanisme étape par étape

| Étape | Action | T-Code / Fiori | Table alimentée | Impact comptable |
|---|---|---|---|---|
| **1** | Demande d'Achat | ME51N · Fiori : Gérer les DA | EBAN | Aucun — document interne |
| **2** | Commande d'Achat (PO) | ME21N · Fiori : Gérer les PO | EKKO / EKPO | Engagement juridique — pas d'écriture FI |
| **3** | Réception marchandise | MIGO · Fiori : Gérer les entrées marchandise | MKPF / MSEG | Débit 3xxxxx ou 6xxxxx · Crédit 408100 (GR/IR) |
| **4** | Saisie facture fournisseur | MIRO · Fiori : Créer facture fournisseur | RBKP / RSEG | Débit 408100 + Débit 44566 · Crédit 401xxx |
| **5** | [Blocage R si écart] | MRBR · Fiori : Libérer factures bloquées | RBKP (statut blocage) | Pièce comptabilisée mais paiement suspendu |
| **6** | Paiement fournisseur | F110 (auto) / F-53 (manuel) | REGUH / REGUP | Débit 401xxx · Crédit 512xxx (banque) |

### 1.2 Tables SAP alimentées — P2P

| Table SAP | Nom fonctionnel | Alimenté par / Contenu |
|---|---|---|
| **EBAN** | Demande d'Achat | Chaque ligne de DA : article, quantité, imputation, statut approbation |
| **EKKO** | En-tête Commande | PO : numéro, fournisseur, société, date, conditions de paiement |
| **EKPO** | Postes Commande | Lignes PO : article, quantité, prix, type d'imputation (K, A…) |
| **MKPF / MSEG** | Document matériel MIGO | Mouvement de stock : type mouvement (101 entrée, 102 retour…), quantité, valeur |
| **RBKP / RSEG** | Document MIRO | En-tête et postes facture LIV avant transfert vers FI |
| **BKPF / BSEG** | Pièce comptable FI | La pièce FI générée par MIRO et le paiement — source de vérité comptable |
| **ACDOCA** | Universal Journal S/4HANA | Remplace BSEG + CO + ML en une table unique — chaque écriture P2P y est tracée |

### 1.3 Le Compte Sentinelle 408100 — GR/IR Clearing

Le compte 408100 (Fournisseurs — Factures à recevoir) est l'intermédiaire entre la réception physique (MIGO) et la réception administrative (MIRO). Son solde est le baromètre de la santé du P2P.

| Situation | Solde 408100 | Diagnostic |
|---|---|---|
| MIGO faite, MIRO pas encore | Créditeur | Normal en cours de période — à résorber avant clôture |
| MIRO faite, MIGO pas encore | Débiteur | Alerte : facture reçue sans marchandise — vérification urgente |
| MIGO + MIRO correspondantes | Solde zéro | P2P propre — cycle terminé |
| Solde chroniquement gonflé | Non nul en fin de période | P2P cassé — tracer via MR11 ou FBL3N sur 408100 |

> [!IMPORTANT]
> **KPI P2P :** Solde du compte 408100 en fin de période. Un compte GR/IR non nul en clôture = provisions obligatoires ou anomalie de flux à traiter avant bilan.

---

## 2. O2C — Order-to-Cash : Vendre et Encaisser

Périmètre : tout ce qui concerne l'entrée d'argent suite à une prestation ou une vente. Le Q2C (Quote-to-Cash) ajoute l'étape de devis en amont — le mécanisme FI reste identique.

### 2.1 Mécanisme étape par étape

| Étape | Action | T-Code / Fiori | Table alimentée | Impact comptable |
|---|---|---|---|---|
| **1** | Devis (Q2C) | VA21 · Fiori : Créer les devis | VBAK / VBAP | Aucun — offre commerciale |
| **2** | Commande Client | VA01 · Fiori : Créer commandes client | VBAK / VBAP | Blocage crédit si encours > plafond FSCM |
| **3** | Livraison | VL01N · Fiori : Créer livraisons sortantes | LIKP / LIPS | Sortie de stock : Débit 607xxx · Crédit 3xxxxx |
| **4** | Facturation SD | VF01 · Fiori : Créer documents de facturation | VBRK / VBRP | Débit 411xxx · Crédit 706xxx + 44571 |
| **5** | Relance (Dunning) | F150 · Fiori : Exécuter les relances | MHNK | Aucun comptable — courrier généré |
| **6** | Encaissement client | F-28 · Fiori : Saisir les paiements entrants | BKPF / BSEG | Débit 512xxx · Crédit 411xxx — PNS → PS |

### 2.2 Hiérarchie des Types de Pièces — RV vs DR

| Type | Libellé | Origine / Contexte | Traçabilité / Impact |
|---|---|---|---|
| **RV** | Facture SD | Générée automatiquement par VF01 depuis une commande client SD | Lien traçable VBAK → VBRK → BKPF. Audit complet de la commande au paiement |
| **DR** | Facture FI directe | Saisie manuelle via FB70 — sans commande SD | Pièce FI uniquement dans BKPF/BSEG. Traçabilité réduite — à éviter |
| **DZ** | Paiement entrant | F-28 ou F110 | Pièce de compensation — lettre les postes AR ouverts (411xxx) |
| **DA** | Avoir client FI | FB75 — avoir direct sans SD | Pièce FI uniquement |
| **RE** | Avoir SD (retour) | VF01 type RE — avoir depuis retour marchandise | Lien VBRK type RE → BKPF. Traçabilité complète incluant le flux retour logistique |

> [!NOTE]
> **Expert Key User O2C :** Toujours privilégier le flux **RV** (VF01 depuis commande SD) plutôt que **DR** (FB70 direct). Une facture DR ne peut pas être rapprochée à une commande SD en reporting — elle casse la chaîne de valeur du O2C.

> [!IMPORTANT]
> **KPI O2C :** DSO (Days Sales Outstanding) = (Encours clients / CA) × 365. Un DSO élevé signifie que les clients paient tard — piloter via F1836 (Analyser les créances — aging AR).

---

## 3. R2R — Record-to-Report : Clôture et Bilan

Périmètre : le processus qui transforme toutes les transactions quotidiennes (P2P, O2C, D2P) en états financiers légaux. C'est le processus transversal — il consolide tous les autres.

### 3.1 Mécanisme étape par étape

| Étape | Action | T-Code / Fiori | Table alimentée | Impact comptable |
|---|---|---|---|---|
| **1** | Contrôle des périodes | OB52 · Fiori : Gérer les périodes comptables | T001B | Ouvre ou ferme les périodes par société et type de compte |
| **2** | Cutoff P2P : nettoyage GR/IR | MR11 · Fiori : Gérer les provisions EM/EF | BKPF / BSEG | Régularise les écarts 408100 en fin de période |
| **3** | Lettrage / Clearing | F-44 (AP) / F-32 (AR) / F.13 (auto) | BSAS / BSAK / BSAD | Transformation PNS → PS |
| **4** | Provisions / Écritures OD | FB50 / FB01 · Fiori : Créer pièces comptables | BKPF / BSEG / ACDOCA | Charges à payer, produits à recevoir |
| **5** | Valorisation devises | FAGL_FC_VAL · Fiori : Exécuter la valorisation | ACDOCA | Ajustement des postes en devises étrangères au cours de clôture |
| **6** | Amortissements FI-AA | AFAB · Fiori : Exécuter l'amortissement | ANEK / ANEA | Débit 681xxx (dotation) · Crédit 281xxx (amort. cumulée) |
| **7** | Réconciliation FI-AA | S_ALR_87011963 + FAGLB03 | ANLC / FAGLFLEXT | Balance immos = comptes collectifs classe 2 du Grand Livre |
| **8** | Clôture analytique CO | KSU5 (répartition) / KSV5 | COSS / COSP | Distribution des charges entre centres de coûts et CO-PA |
| **9** | Balance de vérification | FAGLB03 · Fiori : Afficher les soldes G/L | FAGLFLEXT | Réconciliation grand livre vs balances auxiliaires AP/AR |
| **10** | Fermeture de période | OB52 (fermer) · Fiori : Gérer les périodes | T001B | Aucune écriture supplémentaire possible pour les users standard |

### 3.2 OB52 — Gestion des Périodes Comptables

OB52 gère l'ouverture et la fermeture des périodes dans la table **T001B**. Le paramétrage repose sur deux colonnes de périodes pour une même société : la **période courante** ouverte à tous les utilisateurs, et une **période résiduelle** réservée aux Key Users et comptables de clôture via un groupe d'autorisation dédié. Ce mécanisme permet à l'équipe finance de finaliser les écritures du mois précédent (dotations, provisions) sans exposer la période à l'ensemble des utilisateurs métier.

> [!NOTE]
> **Variante d'exercice K4 vs K0 :** K4 = calendrier standard (période 1 = janvier). K0 = exercice décalé (ex : 01/04 au 31/03 — période 1 = avril). Toujours vérifier la variante via OB29/OB37 avant toute intervention sur OB52 dans un nouveau client.

### 3.3 Le Cockpit de Clôture — CLOCOC / Fiori

En S/4HANA, la clôture peut être orchestrée via le **Closing Cockpit** (transaction `CLOCOC` ou application Fiori dédiée). C'est l'approche moderne qui remplace la gestion manuelle tâche par tâche.

| Fonctionnalité | Détail |
|---|---|
| **Modèle de clôture** | Template avec toutes les tâches R2R ordonnées (MR11, AFAB, KSU5…) et leurs dépendances |
| **Planification et suivi** | Chaque tâche est assignée à un responsable avec une date limite — tableau de bord en temps réel |
| **Exécution en chaîne** | Lancement automatique des jobs en arrière-plan dès que la tâche précédente est validée |
| **Traçabilité** | Historique complet des exécutions, statuts (OK / KO / En cours), journaux d'erreurs par tâche |
| **Intégration Fiori** | Vue consolidée accessible depuis le navigateur — pas besoin de SAP GUI pour le pilotage |

> [!TIP]
> Le Closing Cockpit est particulièrement valorisant à mentionner en entretien — il démontre une maîtrise du R2R au-delà de la simple exécution des transactions, en positionnant le consultant comme pilote du processus de clôture.

### 3.4 Cutoff P2P / R2R — MR11 et Provision 408100

| Situation 408100 | Action MR11 | Écriture générée |
|---|---|---|
| MIGO faite, MIRO en attente | Provision automatique | Débit 6xxxxx (charge) · Crédit 408100 |
| MIRO faite, MIGO jamais reçue | Alerte manuelle | À annuler via MR8M ou tracer avec le service achat |
| Résidus anciens (> 3 mois) | MR11 force la régularisation | Écriture de régularisation sur le compte de charge ou produit |

> [!TIP]
> Lancer MR11 systématiquement à J-2 avant la fermeture de période. Un 408100 non nettoyé s'accumule mois après mois et devient difficile à justifier en audit.

### 3.5 Réconciliation FI-AA — Au Centime Près

| Outil | Ce qu'il affiche | Source de données |
|---|---|---|
| **S_ALR_87011963** | Valeur brute, amortissements cumulés, valeur nette — par classe d'immo | FI-AA : tables ANLC et ANEA |
| **FAGLB03** | Soldes comptes G/L classe 2 : 211xxx, 213xxx, 281xxx | Grand Livre : FAGLFLEXT / ACDOCA |
| **AO90** | Mapping classes d'immo FI-AA → comptes G/L classe 2 | Customizing SPRO — source des divergences si mal paramétré |

### 3.6 Table ACDOCA — Le Universal Journal S/4HANA

| Dimension | ECC (table source) | S/4HANA ACDOCA |
|---|---|---|
| Écriture FI | BKPF + BSEG | Une ligne ACDOCA — champs RBUKRS, RACCT, HSL |
| Analytique CO | COSP + COSS | Même ligne ACDOCA — champs RCNTR, PRCTR |
| Valorisation ML | MLCD (séparée) | Intégré dans ACDOCA — réconciliation FI/CO disparaît |

> [!IMPORTANT]
> **KPI R2R :** Fast Close — objectif J+3 à J+5. Les trois actions critiques : (1) MR11 nettoyé, (2) S_ALR_87011963 = FAGLB03 validé, (3) balances AP/AR réconciliées avec FAGLB03.

---

## 4. D2P — Design-to-Produce : Fabriquer

Périmètre : concevoir un produit et le fabriquer. Le D2P est le processus le plus lié aux modules PP (Production Planning) et CO-PC (Product Costing).

### 4.1 Mécanisme étape par étape

| Étape | Action | T-Code / Fiori | Table alimentée | Impact comptable |
|---|---|---|---|---|
| **1** | Nomenclature (BOM) | CS01 / CS03 · Fiori : Gérer les nomenclatures | MAST / STPO | Aucun — structure théorique du produit |
| **2** | Gamme de fabrication | CA01 · Fiori : Gérer les gammes | CRHD / PLPO | Aucun — temps et ressources théoriques |
| **3** | Calcul du coût standard | CK11N + CK24 | CKIS / MBEW | Prix standard (type S) affecté sur la fiche article |
| **4** | Ordre de Fabrication (OF) | CO01 · Fiori : Créer ordres de production | AUFK / AFPO | Engagement : prévision des coûts de l'ordre |
| **5** | Consommation composants | MIGO mv. 261 / CO11N | MSEG / COEP | Débit compte coût OF · Crédit stock composants |
| **6** | Entrée en stock PF | MIGO mv. 101 depuis OF | MSEG / MBEW | Débit 3xxxxx stock PF · Crédit compte OF |
| **7** | Solde / Écarts CO-PC | KO88 / CO88 (en masse) | COSS / COSP | Écart = coût réel − coût standard → compte 635xxx |

### 4.2 Analyse des Écarts — Le Juge de Paix D2P

| Type d'écart | Cause | Compte PCG | Action Key User |
|---|---|---|---|
| Écart quantité | Plus de matière consommée que prévu dans la BOM | 635100 | Revoir la BOM ou investiguer les rebuts |
| Écart prix | Matière achetée à un prix différent du standard | 635200 | Revoir le prix standard ou la négociation achat |
| Écart temps | Temps opératoire réel différent de la gamme théorique | 635300 | Revoir la gamme ou optimiser le process de production |

> [!IMPORTANT]
> **KPI D2P :** Écart sur coût de revient = (Coût réel OF − Coût standard) / Coût standard × 100. Tables à surveiller : COSS vs COSP via S_ALR_87013127.

---

## 5. H2R — Hire-to-Retire : Gérer l'Humain

Périmètre : le cycle de vie complet du collaborateur, du recrutement à la retraite. L'interface critique pour un consultant FICO est le transfert Paie → Comptabilité.

### 5.1 Mécanisme étape par étape

| Étape | Action | T-Code / Fiori | Table alimentée | Impact comptable |
|---|---|---|---|---|
| **1** | Recrutement | PA40 · SuccessFactors RCM | PA0001 | Aucun comptable — création du dossier RH |
| **2** | Gestion des temps | CAT2 / PT60 | CATSDB / PTEX2000 | Base de calcul de la paie (heures, absences, congés) |
| **3** | Calcul de la paie | PC00_M99_CALC / PY | RT / PC261 | Calcul brut, net, cotisations — pas encore en FI |
| **4** | Transfert paie → FI | PC00_M99_CIPE | BKPF / BSEG / ACDOCA | Débit 641xxx (salaires bruts) · Crédit 431xxx + 421xxx |
| **5** | Paiement salaires | F110 · RFFOD__S | REGUH / REGUP | Débit 421xxx (net à payer) · Crédit 512xxx (banque) |
| **6** | Départ / Retraite | PA30 + calcul solde | PA0000 / PA0019 | Provisions congés payés, précomptes de solde |

### 5.2 L'Interface Paie → FI : Point de Vigilance FICO

| Élément de paie | Compte G/L (PCG FR) | Type de compte | Contrôle Key User |
|---|---|---|---|
| Salaires bruts | 641100 — Salaires | Charge P&L | Débit 641xxx = masse salariale brute |
| Cotisations patronales | 645xxx — Charges sociales | Charge P&L | Vérifier cohérence avec DSN |
| Cotisations à payer (URSSAF…) | 431xxx — Organismes sociaux | Bilan passif | PNS à solder à la date d'échéance URSSAF |
| Net à payer salariés | 421xxx — Personnel rémunérations dues | Bilan passif | PNS soldé le jour du virement F110 |
| Provision congés payés | 641200 / 428100 | Charge / Bilan passif | KPI H2R : justesse de la provision CP en clôture |

> [!IMPORTANT]
> **KPI H2R :** Justesse de la provision pour congés payés (428100). En clôture annuelle, ce compte doit refléter exactement le nombre de jours de CP acquis non pris × salaire journalier. Un écart > 5% entre la provision comptabilisée et le calcul RH est une anomalie à justifier pour le commissaire aux comptes.

---

## 🏢 À propos

**Auteur :** Moez L'Agha — Consultant SAP FI/CO  
**Structure :** MTC Computing (SIREN 820 341 558)  
**Contact :** [LinkedIn](https://www.linkedin.com/in/) · [SAP Community](https://community.sap.com/)

> Ce document fait partie d'une série de référentiels SAP FICO publiés sous la marque MTC Computing.
> Reproduction à des fins pédagogiques autorisée avec mention de la source.
> Prochain document : **AuC — Immobilisations en cours et Clôture FI-AA**
