# OpenListing

Spécification ouverte et neutre pour décrire une petite annonce (classified ad) de façon
portable entre producteurs (applications de saisie) et consommateurs (plateformes de publication).

**Version courante : `0.1.1`**

## Écosystème actuel

| Rôle | Implémentation |
|---|---|
| Producteur | Insera (insera.ch) — web + backend |
| Producteur | Insera mobile (ex openlisting-publisher, app Android Expo) |
| Consommateur | Joomil — `POST /api/v1/openlisting/import` (my.joomil.ch) |

La spec est volontairement **neutre** : elle n'appartient à aucun produit. Les champs propres
à une destination vivent dans `destinationData.<plateforme>`, jamais au niveau générique.

## Format : batch

Un échange OpenListing est un **batch** JSON (`Content-Type: application/openlisting+json`) :

```json
{
  "kind": "openlisting.batch",
  "openlistingVersion": "0.1.1",
  "source": "insera",
  "items": [ { … } ]
}
```

| Champ | Type | Obligatoire | Description |
|---|---|---|---|
| `kind` | string | ✅ | Toujours `openlisting.batch` |
| `openlistingVersion` | string | ✅ | Version de la spec (`0.1.1`) |
| `source` | string | — | Identifiant du producteur (idempotence côté consommateur) |
| `items` | array | ✅ | 1 à 20 items |

## Item `classified_ad`

```json
{
  "id": "insera_42",
  "type": "classified_ad",
  "title": "Vélo de course Cilo vintage",
  "description": "Beau vélo suisse des années 80…",
  "price": { "amount": 280, "currency": "CHF" },
  "condition": "very_good",
  "location": {
    "postalCode": "1700",
    "city": "Fribourg",
    "region": "FR",
    "country": "CH"
  },
  "media": [
    { "position": 0, "uri": "…" }
  ],
  "destinationData": {
    "joomil": {
      "categoryId": 78,
      "adType": "offre",
      "postalCode": "1700",
      "city": "Fribourg",
      "canton": "FR",
      "country": "CH",
      "phone": ""
    }
  }
}
```

| Champ | Type | Obligatoire | Description |
|---|---|---|---|
| `id` | string | ✅ | Identifiant stable côté producteur. Clé d'idempotence `(user, source, id)` : un ré-envoi met à jour l'annonce au lieu de la dupliquer |
| `type` | string | ✅ | `classified_ad` |
| `title` | string | ✅ | Titre de l'annonce |
| `description` | string | ✅ | Description complète |
| `price.amount` | number | ✅ | Montant |
| `price.currency` | string | ✅ | ISO 4217 (`CHF` seul supporté par Joomil aujourd'hui) |
| `condition` | string | — | Enum canonique ci-dessous |
| `location` | object | — | Localisation structurée générique |
| `media` | array | — | Descripteurs de photos ordonnés. Le transfert binaire est hors spec (upload séparé chez le consommateur) |
| `destinationData` | object | — | Un objet par plateforme cible, clés libres propres à chaque destination |

## Enum canonique `condition`

Sur le fil, les valeurs sont **anglaises et canoniques** ; chaque application mappe vers ses
libellés d'affichage localisés :

`unknown` · `new` · `like_new` · `very_good` · `good` · `used` · `for_parts`

## Règles de versionnement

- La version est portée par le batch (`openlistingVersion`), jamais par item.
- Un consommateur rejette explicitement une version non supportée (`unsupported_openlisting_version`).
- Ajouts de champs optionnels = incrément patch. Changement de champ existant = incrément mineur + période de double support.
- Objectif de stabilité : pas de breaking change tant que la spec est en `0.x` sans nécessité démontrée par deux implémentations réelles.

## Contenu du repo

- `schema/openlisting-batch.schema.json` — JSON Schema (draft-07) du batch
- `examples/batch-minimal.json` — batch valide minimal
- Ce README = la spec lisible ; en cas de divergence, le schema fait foi

## Historique

- **0.1.1** (2026-07) — version en production : import Joomil, producteurs Insera & Insera mobile
- 0.1.0 (2026-05) — MVP initial (openlisting-publisher)
