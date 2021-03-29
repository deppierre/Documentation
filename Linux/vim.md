# Navigation
- Copie/colle
`Y` copie ligne
`p` ou `P` colle la ligne
- remplacer 
  - `r`: remplace un caractère
  - `cw` pour remplacer le mot
- recherche :
  - `/string string` : chercher string en avant
  - `?string string?` : chercher string précédement (aller d'abord en bas du fichier)
    - `n` next occurence suivante
    - `N` previous occurence precedente
  - `g/motif/s//motif2/g`
- suppression
  - `x` un caractère
  - `dd` supprime ligne
  - `dw` supprime mot
- insertion :
  - `a` : insere à droit du mot
  - `A` : insere a la fin de la ligne
- afficher lignes: `* :set nu`
