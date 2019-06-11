## manipulation de tables & visualisation sous R
### (avec tidyverse)

Frédéric Mahé, Etienne Loire, Florentin Constancias

6 juin 2019


<!-- ----------------------------------------------------------------

                            Prologue

    ----------------------------------------------------------------
-->

source:

https://github.com/frederic-mahe/tidyverse-fanclub-Montpellier-2019



## R & R Studio
#### born to rock

Note: notes réservées au mode présentation.


![R logo](https://www.r-project.org/Rlogo.png) <!-- .element height="20%" width="20%" -->

- 1993
- logiciel libre
- multi-plateforme
- modulaire et en forte croissance


![Scholarly Impact 2018](https://i0.wp.com/r4stats.com/wp-content/uploads/2017/06/Fig_2d_ScholarlyImpact2016.png)

popularité croissante

Note: SPSS à partir de 100 €/personne/mois


![R_packages_growth](./images/R_packages_growth.png)

fonctionnalités croissantes
([code](https://blog.revolutionanalytics.com/2016/04/cran-package-growth.html))


![Hadley Wickham](https://policyviz.com/wp-content/uploads/2017/01/Wickham_Banner-1140x700.png)

Hadley Wickham, the Man Who Revolutionized R


[R Studio](https://www.rstudio.com/)

Interface graphique pour R (et bien plus)

![R Studio](https://www.rstudio.com/wp-content/uploads/2014/06/RStudio-Ball.png) <!-- .element height="20%" width="20%" -->


### manipulation (unifiée) et visualisation de données

[grammar of graphics](https://www.springer.com/us/book/9780387245447) par Leland Wilkinson

- Chief Scientist for R Studio,
- ggplot2,
- dplyr,
- tidyr,
- et beaucoup d'autres,


### Objectifs

gagner en lisibilité et standardiser:

``` R
## R base
d[, function3(function2(function1(d$var == 0)))]

# tidy
d %>%
    filter(var == 0) %>%
    function1() %>%
    function2() %>%
    function3()
```



### Cleaning and visualizing genomic data

[a case study in tidy analysis](http://varianceexplained.org/r/tidy-genomics/)

![](http://varianceexplained.org/images/david_robinson_picture2.jpg)

David Robinson


## Growth Rate, Cell Cycle, Stress Response, and Metabolic Activity in Yeast

[Brauer et al. (2008)](https://doi.org/10.1091/mbc.E07-08-0779)

[téléchargez-moi !](http://varianceexplained.org/files/Brauer2008_DataSet1.tds)


## Premières commandes avec R Studio

``` R
library(tidyverse)

## load
read_tsv("Brauer2008_DataSet1.tds") -> Brauer2008_DataSet1

## inspect (different commands)
View(Brauer2008_DataSet1)
dim(Brauer2008_DataSet1)
glimpse(Brauer2008_DataSet1)
Brauer2008_DataSet1
```

nombre de colonnes et de lignes ?


## Description des données

6 milieux de culture en déficit d'un nutriment :

- glucose (G)
- ammonium (N)
- phosphate (P)
- sulfate (S)
- leucine (L)
- uracil (U)

6 vitesses de croissance: lent (0,05) à rapide (0,3)


## Disposition des données : pourquoi changer ?

Les outils d'Hadley Wickham réclament :

- une seule variable par colonne,
- une seule observation par ligne,
- un seul type d'observation par tableau

Ce qui ne va pas dans la disposition actuelle :

- G0.05 n'est pas un nom de variable,
- plusieurs variables dans la colonne NAME


## Séparer une colonne en plusieurs variables

La colonne NAME contient :

- un nom de gène (ex : SFB2) ou rien,
- un nom de processus biologique,
- une fonction moléculaire,
- un identifiant,
- un autre identifiant (?)

Séparer les variables facilite l'exploration et permet de poser des
questions plus précises.


## Séparer une colonne en plusieurs variables

fonction _separate_ de tidyr :

``` R
new_cols <- c("name", "BP", "MF", "systematic_name", "number")
Brauer2008_DataSet1 %>%
    separate(NAME, new_cols, sep = "[|]{2}")
```


``` text
# A tibble: 5,537 x 44
   GID   YORF  name  BP    MF    systematic_name number GWEIGHT G0.05  G0.1
   <chr> <chr> <chr> <chr> <chr> <chr>           <chr>    <int> <dbl> <dbl>
 1 GENE… A_06… "SFB… " ER… " mo… " YNL049C "     " 108…       1 -0.24 -0.13
 2 GENE… A_06… ""    " bi… " mo… " YNL095C "     " 108…       1  0.28  0.13
 3 GENE… A_06… "QRI… " pr… " me… " YDL104C "     " 108…       1 -0.02 -0.27
 4 GENE… A_06… "CFT… " mR… " RN… " YLR115W "     " 108…       1 -0.33 -0.41
 5 GENE… A_06… "SSO… " ve… " t-… " YMR183C "     " 108…       1  0.05  0.02
 6 GENE… A_06… "PSP… " bi… " mo… " YML017W "     " 108…       1 -0.69 -0.03
 7 GENE… A_06… "RIB… " ri… " ps… " YOL066C "     " 108…       1 -0.55 -0.3
 8 GENE… A_06… "VMA… " va… " hy… " YPR036W "     " 108…       1 -0.75 -0.12
 9 GENE… A_06… "EDC… " de… " mo… " YEL015W "     " 108…       1 -0.24 -0.22
10 GENE… A_06… "VPS… " pr… " pr… " YOR069W "     " 108…       1 -0.16 -0.38
```
il y a des espaces en trop dans nos nouvelles colonnes...


vérification :
``` R
Brauer2008_DataSet1 %>%
    separate(NAME, new_cols, sep = "[|]{2}") %>%
    select(BP) %>%
    head()
```

``` text
1 " ER to Golgi transport "
2 " biological process unknown "
3 " proteolysis and peptidolysis "
4 " mRNA polyadenylylation* "
5 " vesicle fusion* "
6 " biological process unknown "
```


la commande _join_ est le complément de _separate_

beaucoup de commandes _tidyverse_ ont une commande complémentaire, par
exemple _gather_ est le complément de _spread_


## Élimination des espaces (simple)

fonction _trimws_ (trim whitespace) :

``` R
Brauer2008_DataSet1 %>%
    separate(NAME, new_cols, sep = "[|]{2}") %>%
    mutate(name = trimws(BP)) %>%
    select(BP) %>%
    head()
```
``` text
1 ER to Golgi transport
2 biological process unknown
3 proteolysis and peptidolysis
4 mRNA polyadenylylation*
5 vesicle fusion*
6 biological process unknown
```


## Élimination des espaces (multiple)

fonction _trimws_ (trim whitespace) :

``` R
Brauer2008_DataSet1 %>%
    separate(NAME, new_cols, sep = "[|]{2}") %>%
    mutate_at(vars(name:systematic_name), funs(trimws)) %>%
    select(BP) %>%
    head()
```

``` text
1 ER to Golgi transport
2 biological process unknown
3 proteolysis and peptidolysis
4 mRNA polyadenylylation*
5 vesicle fusion*
6 biological process unknown
```


## Élimination de colonnes

``` R
Brauer2008_DataSet1 %>%
    separate(NAME, new_cols, sep = "[|]{2}") %>%
    mutate_at(vars(name:systematic_name), funs(trimws)) %>%
    select(-number, -GID, -YORF, -GWEIGHT)
```
``` text
# A tibble: 5,537 x 40
   name  BP    MF    systematic_name G0.05  G0.1 G0.15  G0.2 G0.25  G0.3 N0.05
   <chr> <chr> <chr> <chr>           <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
 1 SFB2  ER t… mole… YNL049C         -0.24 -0.13 -0.21 -0.15 -0.05 -0.05  0.2 
 2 ""    biol… mole… YNL095C          0.28  0.13 -0.4  -0.48 -0.11  0.17  0.31
 3 QRI7  prot… meta… YDL104C         -0.02 -0.27 -0.27 -0.02  0.24  0.25  0.23
 4 CFT2  mRNA… RNA … YLR115W         -0.33 -0.41 -0.24 -0.03 -0.03  0     0.2 
 5 SSO2  vesi… t-SN… YMR183C          0.05  0.02  0.4   0.34 -0.13 -0.14 -0.35
 6 PSP2  biol… mole… YML017W         -0.69 -0.03  0.23  0.2   0    -0.27  0.17
 7 RIB2  ribo… pseu… YOL066C         -0.55 -0.3  -0.12 -0.03 -0.16 -0.11  0.04
 8 VMA13 vacu… hydr… YPR036W         -0.75 -0.12 -0.07  0.02 -0.32 -0.41  0.11
 9 EDC3  dead… mole… YEL015W         -0.24 -0.22  0.14  0.06  0    -0.13  0.3 
10 VPS5  prot… prot… YOR069W         -0.16 -0.38  0.05  0.14 -0.04 -0.01  0.39
```


## Passage au format long

Beaucoup de commandes tidyverse nécessitent une table au _format
long_ : une variable donnée ne doit être présente que dans une seule
colonne.


``` R
Brauer2008_DataSet1 %>%
    separate(NAME, new_cols, sep = "[|]{2}") %>%
    mutate_at(vars(name:systematic_name), funs(trimws)) %>%
    select(-number, -GID, -YORF, -GWEIGHT) %>%
    names()
```

``` text
 [1] "name"            "BP"              "MF"              "systematic_name"
 [5] "G0.05"           "G0.1"            "G0.15"           "G0.2"
 [9] "G0.25"           "G0.3"            "N0.05"           "N0.1"
[13] "N0.15"           "N0.2"            "N0.25"           "N0.3"
[17] "P0.05"           "P0.1"            "P0.15"           "P0.2"
[21] "P0.25"           "P0.3"            "S0.05"           "S0.1"
[25] "S0.15"           "S0.2"            "S0.25"           "S0.3"
[29] "L0.05"           "L0.1"            "L0.15"           "L0.2"
[33] "L0.25"           "L0.3"            "U0.05"           "U0.1"
[37] "U0.15"           "U0.2"            "U0.25"           "U0.3"
```

La variable _Glucide_ est présente dans plusieurs colonnes. Il faut
corriger ça.


## Passage au format long

objectif : une colonne par variable. Passage de 36 colonnes à 3
(nutriment, taux de croissance, expression) avec la fonction _gather_
de tidyr :

``` R
Brauer2008_DataSet1 %>%
    separate(NAME, new_cols, sep = "[|]{2}") %>%
    mutate_at(vars(name:systematic_name), funs(trimws)) %>%
    select(-number, -GID, -YORF, -GWEIGHT) %>%
    gather(sample, expression, G0.05:U0.3)
```


## Passage au format long

``` text
# A tibble: 199,332 x 6
   name  BP              MF                   systematic_name sample expression
   <chr> <chr>           <chr>                <chr>           <chr>       <dbl>
 1 SFB2  ER to Golgi tr… molecular function … YNL049C         G0.05       -0.24
 2 ""    biological pro… molecular function … YNL095C         G0.05        0.28
 3 QRI7  proteolysis an… metalloendopeptidas… YDL104C         G0.05       -0.02
 4 CFT2  mRNA polyadeny… RNA binding          YLR115W         G0.05       -0.33
 5 SSO2  vesicle fusion* t-SNARE activity     YMR183C         G0.05        0.05
 6 PSP2  biological pro… molecular function … YML017W         G0.05       -0.69
 7 RIB2  riboflavin bio… pseudouridylate syn… YOL066C         G0.05       -0.55
 8 VMA13 vacuolar acidi… hydrogen-transporti… YPR036W         G0.05       -0.75
 9 EDC3  deadenylylatio… molecular function … YEL015W         G0.05       -0.24
10 VPS5  protein retent… protein transporter… YOR069W         G0.05       -0.16
```
_sample_ contient encore deux variables.


## Séparation des variables nutriment et taux de croissance

fonction _separate_ de tidyr :

``` R
Brauer2008_DataSet1 %>%
    separate(NAME, new_cols, sep = "[|]{2}") %>%
    mutate_at(vars(name:systematic_name), funs(trimws)) %>%
    select(-number, -GID, -YORF, -GWEIGHT) %>%
    gather(sample, expression, G0.05:U0.3) %>%
    separate(sample, c("nutrient", "rate"),
             sep = 1, convert = TRUE)
```


## Séparation des variables nutriment et taux de croissance

``` text
# A tibble: 199,332 x 7
   name  BP           MF              systematic_name nutrient  rate expression
   <chr> <chr>        <chr>           <chr>           <chr>    <dbl>      <dbl>
 1 SFB2  ER to Golgi… molecular func… YNL049C         G         0.05      -0.24
 2 ""    biological … molecular func… YNL095C         G         0.05       0.28
 3 QRI7  proteolysis… metalloendopep… YDL104C         G         0.05      -0.02
 4 CFT2  mRNA polyad… RNA binding     YLR115W         G         0.05      -0.33
 5 SSO2  vesicle fus… t-SNARE activi… YMR183C         G         0.05       0.05
 6 PSP2  biological … molecular func… YML017W         G         0.05      -0.69
 7 RIB2  riboflavin … pseudouridylat… YOL066C         G         0.05      -0.55
 8 VMA13 vacuolar ac… hydrogen-trans… YPR036W         G         0.05      -0.75
 9 EDC3  deadenylyla… molecular func… YEL015W         G         0.05      -0.24
10 VPS5  protein ret… protein transp… YOR069W         G         0.05      -0.16
```
tidyverse a deviné que _rate_ contient des nombres.


## Focus sur le gène LEU1

fonction _filter_ de dplyr :

``` R
Brauer2008_DataSet1 %>%
    separate(NAME, new_cols, sep = "[|]{2}") %>%
    mutate_at(vars(name:systematic_name), funs(trimws)) %>%
    select(-number, -GID, -YORF, -GWEIGHT) %>%
    gather(sample, expression, G0.05:U0.3) %>%
    separate(sample, c("nutrient", "rate"),
             sep = 1, convert = TRUE) %>%
    filter(name == "LEU1")
```
LEU1 est un gène participant à la synthèse de la leucine. On a 36
observations (6 conditions et 6 taux de croissance).


## Visualisation avec ggplot2

``` R
Brauer2008_DataSet1 %>%
    separate(NAME, new_cols, sep = "[|]{2}") %>%
    mutate_at(vars(name:systematic_name), funs(trimws)) %>%
    select(-number, -GID, -YORF, -GWEIGHT) %>%
    gather(sample, expression, G0.05:U0.3) %>%
    separate(sample, c("nutrient", "rate"),
             sep = 1, convert = TRUE) %>%
    filter(name == "LEU1") %>%
    ggplot(aes(rate, expression, color = nutrient)) +
    geom_line()
```


## ggplot2 ? gnéé ??

grammaire graphique : données + système de coordonnées + géométrie =
graphique

``` R
ggplot(data = cleaned_data,
       aes(x = rate, y = expression, color = nutrient)) +
    geom_line()
```

système de coordonnées cartésien par défaut (polaire, logarithmique,
cartographique, ...)


## Visualisation avec ggplot2

![](http://varianceexplained.org/figs/2015-11-19-tidy-genomics/unnamed-chunk-14-1.png)


## Focus sur une voie de biosynthèse

``` R
Brauer2008_DataSet1 %>%
    separate(NAME, new_cols, sep = "[|]{2}") %>%
    mutate_at(vars(name:systematic_name), funs(trimws)) %>%
    select(-number, -GID, -YORF, -GWEIGHT) %>%
    gather(sample, expression, G0.05:U0.3) %>%
    separate(sample, c("nutrient", "rate"),
             sep = 1, convert = TRUE) %>%
    filter(BP == "leucine biosynthesis") %>%
    ggplot(aes(rate, expression, color = nutrient)) +
    geom_line() +
    facet_wrap(. ~ name)
```


## Visualisation avec ggplot2

![](http://varianceexplained.org/figs/2015-11-19-tidy-genomics/unnamed-chunk-15-1.png)


## Focus sur une voie de biosynthèse

ajustement de droite (modèle linéaire, ou _lm_)

``` R
Brauer2008_DataSet1 %>%
    separate(NAME, new_cols, sep = "[|]{2}") %>%
    mutate_at(vars(name:systematic_name), funs(trimws)) %>%
    select(-number, -GID, -YORF, -GWEIGHT) %>%
    gather(sample, expression, G0.05:U0.3) %>%
    separate(sample, c("nutrient", "rate"),
             sep = 1, convert = TRUE) %>%
    filter(BP == "leucine biosynthesis") %>%
    ggplot(aes(rate, expression, color = nutrient)) +
    geom_point() +
    geom_smooth(method = "lm", se = FALSE) +
    facet_wrap(. ~ name)
```


## Visualisation avec ggplot2

![](http://varianceexplained.org/figs/2015-11-19-tidy-genomics/unnamed-chunk-16-1.png)


## Focus sur le métabolisme du soufre

``` R
Brauer2008_DataSet1 %>%
    separate(NAME, new_cols, sep = "[|]{2}") %>%
    mutate_at(vars(name:systematic_name), funs(trimws)) %>%
    select(-number, -GID, -YORF, -GWEIGHT) %>%
    gather(sample, expression, G0.05:U0.3) %>%
    separate(sample, c("nutrient", "rate"),
             sep = 1, convert = TRUE) %>%
    filter(BP == "sulfur metabolism") %>%
    ggplot(aes(rate, expression, color = nutrient)) +
    geom_point() +
    geom_smooth(method = "lm", se = FALSE) +
    facet_wrap(. ~ name + systematic_name, scales = "free_y")
```


## Visualisation avec ggplot2

![](http://varianceexplained.org/figs/2015-11-19-tidy-genomics/unnamed-chunk-17-1.png)


## Script complet

``` R
library(tidyverse)

new_cols <- c("name", "BP", "MF", "systematic_name", "number")

## load and clean
read_tsv("Brauer2008_DataSet1.tds") %>%
    separate(NAME, new_cols, sep = "[|]{2}") %>%
    mutate_at(vars(name:systematic_name), funs(trimws)) %>%
    select(-number, -GID, -YORF, -GWEIGHT) %>%
    gather(sample, expression, G0.05:U0.3) %>%
    separate(sample, c("nutrient", "rate"),
             sep = 1, convert = TRUE) -> cleaned_data

## focus and visualize
cleaned_data %>%
    filter(BP == "sulfur metabolism") %>%
    ggplot(aes(rate, expression, color = nutrient)) +
    geom_point() +
    geom_smooth(method = "lm", se = FALSE) +
    facet_wrap(. ~ name + systematic_name, scales = "free_y")
```


## Modeling gene expression with broom

### a case study in tidy analysis

[analyses](http://varianceexplained.org/r/tidy-genomics-broom/)
supplémentaires sur le même jeu de données

par David Robinson



## tidyverse ecosystem


## ggrepel

[ggrepel](https://cran.r-project.org/web/packages/ggrepel): repel
overlapping text labels away from each other

![](http://www.sthda.com/sthda/RDoc/figure/ggplot2/ggplot2-add-text-ggrepel-3.png)


## ggbio

[ggbio](https://bioconductor.org/packages/release/bioc/html/ggbio.html):
visualization of genomic data

![](https://raw.githubusercontent.com/tengfei/ggbio/gh-pages/images/cir.png)


![](https://raw.githubusercontent.com/tengfei/ggbio/gh-pages/images/interval.png)


## ggtree

[ggtree](http://bioconductor.org/packages/release/bioc/html/ggtree.html):
visualization of phylogenetic data

![](https://guangchuangyu.github.io/featured_img/ggtree/2015_peiyu_1-s2.0-S1567134815300721-gr1.jpg)


## ggmap

[ggmap](https://github.com/dkahle/ggmap): plot maps

![](https://raw.githubusercontent.com/dkahle/ggmap/master/tools/README-faceting-1.png)


## ggtern

[ggtern](http://www.ggtern.com): make ternary plots

![](http://www.ggtern.com/wp-content/uploads/2017/07/geom_encircle.png)


## gganimate
  
![gganimate example](https://www.datanovia.com/en/wp-content/uploads/dn-tutorials/r-tutorial/images/shadow_mark.gif)


![](https://www.datanovia.com/en/wp-content/uploads/dn-tutorials/r-tutorial/images/let-data-gradually-appear-show-points.gif)

``` R
p + 
  geom_point() +
  transition_reveal(Day)
```


## ggnetwork
  
![ggnetwork example](http://f.briatte.org/r/images/ggnetwork-network-geometries-for-ggplot2-2.png)


## ggplot2 extensions

visitez la [gallerie](http://www.ggplot2-exts.org/gallery/) pour plus
de ggtrucs !


## Documentation

antisèches [tidyr, dplyr, ggplot2,
RStudio](https://www.rstudio.com/resources/cheatsheets/), tutoriel
[ggplot2](http://r-statistics.co/ggplot2-Tutorial-With-R.html)


### Exemples avancés

![owl](https://cdn-images-1.medium.com/max/1000/1*ps1LThoFav5Joqyd1mPAyg.jpeg) <!-- .element height="50%" width="50%" -->



## Conclusions

Tout est possible

Rien ne va marcher

Mais quelqu'un a déjà essayé

<!-- two newlines for a new vertical slide  -->
<!-- three newlines for a new horizontal slide  -->
