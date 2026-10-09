---
title: "Vérifier"
translationKey: verify
url: /fr/verifier/
description: "Tout ce que Tadorne publie est signé par une clé GPG unique, ancrée dans le DNS. Voici comment la récupérer et vérifier une signature."
params:
  eyebrow: "Authenticité"
---

Tout ce que Tadorne publie est signé par une clé GPG ed25519 unique, publiée
à deux endroits indépendants — la zone DNS de `tadorne.org`, sous DNSSEC, et
ce site sous `/.well-known/`. Un attaquant devrait compromettre les deux.

## La clé

| Champ | Valeur |
|---|---|
| Identifiant utilisateur | `Tadorne <contact@tadorne.org>` — principal, utilisé pour la découverte |
| Algorithme | ed25519 (EdDSA), signature + certification |
| Empreinte | voir ci-dessous |

<code class="fingerprint">5420 08E3 53EC A628 B4D6  ACAD 8C03 243A FF69 5BF7</code>

## Récupérer la clé

### Par WKD (Web Key Directory)

La voie la plus simple. GnuPG traduit l'adresse en une URL sous
`https://tadorne.org/.well-known/openpgpkey/` et récupère la clé en TLS :

```console
$ gpg --locate-keys contact@tadorne.org
```

Interroger `contact@tadorne.org`, jamais `tadorne@pm.me`. Les deux sont des
identifiants de cette clé, mais seul `tadorne.org` est une zone contrôlée par
Tadorne : une requête WKD sur l'adresse `pm.me` reçoit sa réponse du
fournisseur de messagerie et renvoie une autre clé, sans rapport. Une clé
obtenue par cette voie ne portera pas l'empreinte ci-dessus.

`--locate-keys` importe également la clé. Pour la récupérer sans l'importer,
utiliser `gpg --locate-external-keys` et examiner d'abord la sortie.

### Par le DNS (OPENPGPKEY, RFC 7929)

La clé est aussi publiée dans un enregistrement `OPENPGPKEY` de la zone
`tadorne.org`, elle-même signée par DNSSEC. Cette voie ne dépend pas du tout
du serveur web :

```console
$ gpg --auto-key-locate clear,dane --locate-keys contact@tadorne.org
```

Pour consulter directement l'enregistrement, la validation DNSSEC étant
signalée par le drapeau `ad` dans la réponse :

```console
$ dig +dnssec OPENPGPKEY \
    $(printf 'contact' | sha256sum | cut -c1-56)._openpgpkey.tadorne.org
```

Un enregistrement `TXT` de la même zone porte l'empreinte en clair, comme
troisième recoupement.

### Toujours confirmer l'empreinte

Quel que soit le chemin par lequel la clé arrive, comparer son empreinte à la
valeur ci-dessus avant de s'y fier :

```console
$ gpg --fingerprint contact@tadorne.org
```

## Si une signature échoue

Un échec de vérification signifie l'une de trois choses : le fichier a été
modifié en transit, la clé détenue n'est pas la clé Tadorne, ou la clé
publiée a été renouvelée. Récupérer la clé par les deux voies ci-dessus, les
comparer, et si elles divergent — ou si l'une d'elles diverge de l'empreinte
de cette page — considérer le téléchargement comme compromis et le signaler à
`contact@tadorne.org`.

Tout renouvellement, révocation ou compromission de clé sera annoncé ici et
dans le [journal](/fr/journal/), signé par la clé remplacée.
