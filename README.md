graph TD
    subgraph "Utilisateur & Profil"
        U[users]
    end

    subgraph "Données & Traitement"
        DS[datasources]
        P[products]
        PRED[predictionsEMDN]
    end

    U --"possède 1..N"--> DS
    U --"possède 1..N"--> P
    DS --"provient de 1"--> P
    P --"a une prédiction (1-to-1)"--> PRED
