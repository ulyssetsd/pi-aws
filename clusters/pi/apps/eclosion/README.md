# Éclosion demo

Static French showcase for an aspiring psychologist, intended for adults 18+.

Public URL: https://eclosion.ulyssetassidis.fr

The page explicitly identifies itself as a demo with no consultations available.
It uses no third-party assets, JavaScript, forms, or analytics. Inline botanical
SVG illustrations and native FAQ disclosure elements keep it self-contained.
Indexing is disabled while it is a demonstration. Professional identity and
credentials must be supplied before turning this into a professional service.

Flux includes this directory from `clusters/pi/apps/kustomization.yaml`.
Kustomize generates the content ConfigMap and rewrites the deployment reference;
editing `index.html` or `default.conf` changes the hash and triggers a rollout.
Nginx listens on port 8080 without root privileges. Traefik exposes the hostname
and cert-manager requests its certificate from `letsencrypt-prod`.

Official resources used for the French crisis information:

- https://3114.fr/
- https://www.service-public.gouv.fr/particuliers/vosdroits/F33954
