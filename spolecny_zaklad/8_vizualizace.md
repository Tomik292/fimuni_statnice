# 8. Vizualizace

> [Základní metriky pro hodnocení kvality vizualizace (efektivita a expresivita)](#expresivita-vs-efektivita), [osm základních vizuálních proměnných](#8-základních-vizuálních-proměnných). Základní vizualizační techniky pro [1D](#1d-data), [2D](#2d-data), [3D (explicitní a implicitní reprezentace povrchu)](#3d-data). [Techniky pro vizualizaci multidimenzionálních dat (paralelní souřadnice, RadViz, scatterplot matrices)](#multivariate-data) a [hierarchických struktur (treemaps, dimensional stacking)](#stromy-grafy-sítě). [Základní třídy interakčních technik (fisheye, perspektivní stěny), specifika aplikace interakčních technik v prostoru samotných dat a v prostoru jejich atributů.](#interakce) (PV251)

## Úvod 

"Displaying a given information using a graphical representation."
- proč?
    - prezentovat data tak, abychom podpořili schopnost adresáta získat vhled
    - doručit informaci
    - pomoci interpretaci dat
    - získat pozornost
- pozor, vizualizace mohou snadno manipulovat adresáta, to nechceme
- dnes důležité pro medicínu, mapy, miproskpii, abstraktní data, ...
## Expresivita vs efektivita
 **Expressiveness** 
 - presents the given information and nothing more. 
- It does not introduce incorrect information into the visualization. Using expressiveness, we therefore measure the concentration of information in a given visualization.
- This metric can be expressed for the given information that we want to display to the user as:
	- the ratio M$_{exp}$ = $\frac{displayed\_information}{ information\_we\_want\_to\_communicate}$
The value of  M$_{exp}$ ranges from 0 upwards. 
If:
-  M$_{exp} = 1$ - expressiveness is ideal.
-  M$_{exp} < 1$ - our visualization is not completely expressive because we display less information than originally intended.
- M$_{exp} > 1$ - we present more information than we should. This is potentially dangerous, possibly displaying incorrect information, may interfere with the interpretation of the basic information we want to communicate.

**Efficiency** - can be defined in a similar way as expressiveness. In this case, we refer to the ratio as M$_{eff}$,  in this case, a bit more complicated.
- The goal is to define a metric that measures the time required for interpretation for small datasets - because rendering small data is usually very fast. 
- As this interpretation time increases, either due to increasing complexity or size of the data, then the M$_{eff}$ value decreases, with emphasis on the time required for rendering.
- M$_{eff}$ is defined as follows: M$_{eff}$ = $\frac{1}{(1 + interpret + render)}$
-  $0 ≤ M_{eff} ≤ 1$
- If the $M_{eff}$ value is small, it means that either the interpretation time or the rendering time is large. 
- When the $M_{eff}$ value is close to one, the time required for interpretation and rendering is very short.
## 8 základních vizuálních proměnných

- **poloha**
    - prostorové rozložení je to první, co z vizualizace vnímáme
    - každý symbol chcmeme namapovat na unikátní pozici, zároveň tam ale nechceme moc volného místa mezi nimi - kolik toho můžeme zobrazit je tedy omezeno i rozlišením displeje, miliony bodů se tam prostě nevejdou
    - záleží, jaké promněnné si vybereme jako osy - př. pokud mezi nimi je lineární vztah, tak v grafu je vidět a v barvě méně
    - škálování os - lineární, logaritmické
    - osy jsou užitečné, i když ne všude musí být
    - poznámky k vnímání hloubky
        - objekty schované za jinými
            - problémy, když chceme interakci na výběr objektů
        - zobrazování labelů
            - musí být vidě, jít přečíst, směřovat ke "kameře"
        - občas lepší víc grafů s různými pohledy než vše do jednoho
        - 3D dává smysl, pokud máme 3D data, jinak většinou spíš ne
- **tvar**
    - měly by být dostatečně rozeznatelné
    - nebavíme se o barvě, orientaci, velikosti, ...
    - šipka, kruh, čtverec, krížek, tečka, **glyphs**
    - většinou pro rozlišení kategorií

_(poloha a tvar jsou zásadní, zbytek je pomocný)_

- **barva**
    - vidíme ji hned
    - definována pomocí hue (odstíny) a saturation (kolik barvy a kolik šedé)
    - dá se použít pro diskrétní i spojité veličiny
    - rainbow scale - problémy: neuniformí, umělé předěly i když v datech nejsou
        - proč se to tedy pořád používá? hezky se na to kouká, poutá to pozornost
        - pokud bychom použili jen jednu barvu a měnili jas, tak se zase blbě čtou jednotlivé hodnoty
        - Zaleží co chceme daty prezentovat
	- Je potřeba definovat color map - které vysvětlí co jaké barva popisuje
- **velikost**
    - společný rámec pomáhá přesnosti vjemu
    - pokud je velikost kódována jako šířka čáry, můžeme použít jenom velmi limitovaný počet možností
    - perspektiva ničí vnímání velikosti
    - dobře se mapuje na intervaly nebo spojité hodnoty
- **jas**
    - vnímání je limitované, škála by měla být lineární
    - lze konbinovat například s tvary pro další rozlišení kategorií
- **orientace**
	- rotace glyphů - je potřeba zvolit sprývný gliph (kruh)
- **textura**
	- can be used as additional combination with color, glyph, orientation
- **pohyb**
	- most commonly used as speed in change, we can change positions but also movement of color, brightness or  size

![8 proměnných](app://c3044c24f470d0763fbb49aecc767c567bbe/C:/Users/tomas/OneDrive%20-%20MUNI/Obsidian/fimuni_statnice/obrazky/8_vizualizace/8vars.png?1716826654480)
## Základní vizualizační techniky -  vizualizace dat v prostoru (spatiotemporal data)
- mohou být generována dopředu nebo za běhu vizualizace
    - podle toho typy vizualizací:
        - pasivní: data jsou vygenerována, vykreslena, uživatel se dívá
        - interaktivní: data jsou vygenerována, vykreslena, uživatel může něco měnit, třeba vybírat, co chce zrovna vidět
        - interactive steering: data jsou generována a kreslena naráz, uživatel může měnit, co se generuje
    - seřazené od nejméně po nejvíce infomativní a náročné
- data jsou hlavní část, ale musíme taky komunikovat s uživatelem, co chce, nejen "něco kreslit"
- data raw nebo preprocessed
- dato se skládá z nezávislých a závislých proměnných
    - nezávislé (= doména) - př. čas, místo
    - závislé (= range) - př. teplota
- ordinální (lze řadit) vs nominální (kategori)
    - **ordináln**í a **nominální ranked** jdou řadit, měli bychom vizualizovat tak, aby šly řadit
    - nominální kategorická řadit nejdou, měli bychom je vizualizovat tak, aby k tomu nevybízely (tj. nepoužívat k jejich odlišení například velikost)
    - škálování
        - existuje absolutní 0?
        - jdou počítat vzdálenosti mezi záznamy?
- příklady reprezentace dat 
    - diskrétní neseřazené - koláčový graf
    - diskrétní seřazené - boxplot
 ![[boxplot.png|350]]
    - spojitá - linechart
![[line-chart-example.svg|300]]
    - 2D spojitá - barvy, plocha ve 3D, vrstevnice
    - 2D matice - hedgehog plot, line integral convolution
    ![[Example-of-an-Hedgehog-plot-from-Iris-Measuring-System.png|300]]
    - 3D flow - streamlines, streamsurfaces
    ![[Untitled.png]]
    - prostorová data - isosurfaces, volume rendering
     ![[Untitled.jpg]]
    - vícedimenzionánlní - paralell coordinates
### 1D data
- osa x, pak buď y nebo třeba barva
- multivariate data - osa x a více hodnot na y
    - juxtaposition - víc grafů vedle sebe
    - superimposition - víc grafů do sebe
### 2D data

- obrázek (fotka, rentgen, ale taky třeba scatterplot ...)
- rubber sheet - surface 
    - každý z 2D bod má výšku, uděláme triangulaci a povrch
- cityscape - 3D bloky na mapě, perspektiva
- scatterplot -  není tam interpolace, další vlastnosti dat barva, tvar, ...
- mapa
- kontury, vrstevnice
- juxtaposition - víc 2D grafů do 3D
- superimposition - víc grafů do sebe - př. reliéf s průhlednými plochami
- taky můžeme z dat udělat 1D a pak zobrazit to
    - freq. histogramy
    - slučování řádků a sloupců v obrázku
        - součet, průměr, odchylka, max, min, ...
### 3D data

- explicitní plochy
    - vrcholy ve 3D, hrany mezi nimi, polygony definované pomocí hran
    - parametrické rovnice definující souřadnice bodů na ploše v prostoru + strategie, jak body spojit
- objemová data (voxely)
    - slicing, potom 2D nebo 1D
    - vrstevnicové plochy - pak explicitní plochy
        - převod pomocí marching cubes
        ![Marching cubes](../obrazky/8_vizualizace/marching_cubes.png)
            - pro každý roh definováno, co se má stát, pokud je hodnota pod/nad prahem
            - problémy - můžeme mít díry v povrchu, zabírá to hodně místa
    - přímo renderujeme objem
        - forward mapping
            - promítneme každý voxel na projekční rovinu a určíme, které pixely budou jak barevné
            - problémy: co s pixely ovlivěnými více voxely, co s pixely na které se nenamapovaly žádné voxely, co s tím, když jsou voxely často namapovány mezi pixely
        - ray casting
        ![Ray casting](../obrazky/8_vizualizace/raycasting.png)
            - vyšleme paprsek z každého pixelu projekční roviny přes objemová data, kde se protíná s jakými hodnotami určí výslednou hodnotu
            - problémy: jak vybrat množství bodů, které budeme samplovat v průběhu paprsku, jak spočítat hodnotu tam, kde často padneme mezi voxely, jak pak hodnoty zkombinovat
        - řešení některých problémů: každý pixel má hodnotu "transparentnost", pro forward mapping mapujeme více pixelů na více voxelů (váženě)
    - resampling hraje důležitou roli
    - cutting planes
    ![Cutting planes](../obrazky/8_vizualizace/cutting_planes.png)
        - pokud není paralelní s nějakou osou, tak musíme řešit mapování voxelů na pixely - nejbližší, vážený průměr z blízkých
        - na specifikaci potřebujeme 6 parametrů, 3 poziční a 3 na normálu
        - varianty - neplacaté řezy, sekvence řezů různých orientací, ortogonální řezy
- implicitní plochy
    - zero contour funkce
    - složité, ale dobrý výsledek
- kombinované techniky
    - řezy + vrstevnicové plochy
    - plochy a markery/glyfy
## Multivariate data

- curse of dimensionality
    - algoritmy na exploraci jsou tím pomalejší, čím víc je dimenzí
    - víc dimenzí $\to$ řídká data, chybějící hodnoty
    - machine learning: čím víc dimenzí, tím víc (exponenciálně) potřebujeme dat
- cíle
    - najít clustery, pravidelnosti a nepravidelnosti
- line based techniky
    - kreslení linechartů přes sebe a vedle sebe
        - přes sebe - nemůže tam být moc čar
    - předchozí čára slouží jako reference pro následující
### RadViz
- It is based on physics, more precisely on Hooke's law, and uses the finding of the equilibrium position of a point for the representation. 
- For an N-dimensional data set, N "anchor" points are placed on the circumference of the circle.
- Data rozložená tak, že se blíží k tomu, k čemu jsou podobná
	- pro různé pořadí kotev to vypadá různě
![RadViz](../obrazky/8_vizualizace/radviz.png)
- The anchors are calculated in a following way:
	- given normalized data vector $D_i = (d_{i, 0}, d_{i, 1},...,d_{i, N-1})$ and a set of unit vectors $A$
	- $A_j$ represents the j-th anchor point, we get the following equilibrium calculation:
	 $\sum_{j=0}^{N-1} (A_j - p)*d_j = 0$
	- Where **p** is the vector for the point in equilibrium.
	- The calculation of p is performed:
	  $p=\frac{\sum_{j=0}^{N-1}(A_j*d_j)}{\sum_{j=0}^{N-1}(d_j)}$
###  Scatterplots
-  It consists of a grid containing scatterplots that has N2 cells, where N is the number of dimensions.
- Therefore, each pair of dimensions is drawn twice – it differs only by rotating the graph by 90 degrees.
- Graphs on the main diagonal are often used to communicate dimension information in the corresponding row / column or to plot a histogram of that dimension. Not in graph below.
![Scatterplot matrix](../obrazky/8_vizualizace/scatterplot_matrix.png)
### Parallel coordinates
- suffering from the scalability problem regarding the number of data items that can be comprehensively displayed.
- This problem can be partially solved by using different techniques, such as density-based visualizations, random sampling of data items, or edge bundling or implement some interactive techniques
- Very important is the order of parallel coordinates, a good solutions can be found using Hough space for every pair of coordinates
![Parallel coords](../obrazky/8_vizualizace/parallel_coors.jpg)
- výběr dimenzí, které chceme zobrazit
	- ranking techniky
        - hodnotí jednotlivé dimenze nebo jejich dvojice
            - typická hodnotící fce př. počet outlierů, corelace mezi dimenzemi, least square error lineární regrese, entropie 
        - feature extraction - PCA, MDS
        - problémy: pokud trochu změníme data a uděláme výběr dimenzí znovu, tak může být úplně jiný

## Hierarchické struktury
Hierarchical structures (often referred to simply as "trees") are some of the most common structures used to store information about the relationships between data. Many visualization techniques were therefore created to display these structures and preserved relationships. These techniques and their algorithms can be divided into two classes:
- **Space-filling** – make maximum use of screen space
- **Non-space-filling** – maximum use of space of the output device is not their main criterion
When studying the techniques for displaying hierarchical structures, we will focus on these two types of algorithms.
### **Space-filling methods**
- As the name suggests, these techniques try to make the most use of available screen space.
- This is often accompanied using juxtapositioning to display relations. The two most common approaches to generating this type of hierarchy are rectangular and radial layouts.
- Among the most popular forms of space-filling layout belong treemaps and their various variants. These are an example of a rectangular space layout.
- Representatives of the second class, i.e., the radial layout, are, for example, the so-called "sunburst displays" (in the shape of the sun). Here, the root of the hierarchy is displayed in the center, and nested ring layers are used to display additional hierarchy layers.

#### **Treemaps**
In their basic variant, the rectangle is recursively divided into segments, alternating the horizontal and vertical division of the rectangle. The division is derived from the population of subtrees at a given level. 
![Treemap](../obrazky/8_vizualizace/treemap.png)
Many variants of treemaps have been developed, such as squarified treemaps (to reduce long, thin rectangles) or nested treemaps (to enhance the hierarchical structure)
#### Radial layout
Methods for displaying hierarchy in radial layout, often referred to as "sunburst displays", have the root of the hierarchy located in the center of the radial display and the individual layers of the hierarchy are represented by concentric rings. Each ring is divided based on the number of nodes in a given level of the hierarchy. 
![Sunburst display|500](../obrazky/8_vizualizace/sunburst_display.png|)

**Using color in hierarchical techniques**
For the aforementioned techniques, as well as for other space-filling techniques, color can be used to highlight many attributes, such as the values of each node (e.g., for classification) or it can enhance the display of hierarchical relationships. Other data properties can be communicated, for example, by using symbols and other markings placed in rectangular or circular segments.
### **Non-space-filling methods**
- One of the most common representations of hierarchical relationships are node-link diagrams. The structure of the organization, family trees, or pairings in tournament matchesare just a few examples of common applic ations of these diagrams. 
- The appearance of these trees is most affected by two factors: the degree of branching (the maximum number of siblings per parent) and the depth (determined by the farthest node from the root).

### Dimensional stacking
The technique of folding dimensions was developed by LeBlanc et al. and aims to map data from discrete N-dimensional space to a 2D image in such a way as to minimize data occlusion while preserving most of the spatial information.
Briefly, the mapping is performed as follows:  
- We start with data of dimension 2N + 1 (for an even number of dimensions it is  necessary to supply an additional default dimension of cardinality 1).  
- Select the final cardinality for each dimension.  
- Select one of the dimensions as a dependent variable. The rest are considered  independent variables.  
- Now we create ordered pairs of independent variables (N pairs) and assign each pair its unique value (called velocity) from 1 to N.  
- The pair corresponding to velocity 1  creates a virtual image whose size corresponds to the cardinality of dimensions (the  first dimension of the pair is oriented horizontally, second vertically). 
- At each position of this virtual image, another virtual image is created that corresponds to the dimensions of velocity 2. 
- Again, the size of this image depends on the cardinality of the corresponding dimensions. 
- This process is repeated until all dimensions are included.
In this way, we achieve that each location in a space with many dimensions has its own unique location in the 2D image that results from the mapping.
![[Pasted image 20240601115240.png|400]]
## Interakce
There are several classes of interaction techniques:
- **Navigation** - the user changes the position of the camera and scales the view (what is mapped to the screen). An example is rotation or zooming.
- **Selection** - identifying a specific object, set of objects or area of interest, to which we then apply certain operations, such as highlighting, deleting, or modification.
- **Filtering** - reduces the size of the data we map onto the screen - by removing records, dimensions, or both.
- **Reconfiguration** - the user can change the way data is mapped to graphical entities or attributes. Also PCA or MDS
- Encoding - the user changes graphic attributes, such as point size or line color. The goal is to reveal various properties of the data.
- **Merge** - the user can use tools to merge different views or objects to display related items.
- **Abstracting / concretization** - level-of-detail modification.
- **Aggregation**
- **Hybrid techniques** - combining some of the above techniques. For example, increasing the screen space that is used to display the detail of certain data, while reducing the space devoted to "uninteresting" data that is displayed to preserve context

Interakce mohou probíhat na úrovni různých prostorů
    - screen space (pixely) - posun, zoom, lupa, ...
    - data value space - výběr, filtrování
    - komponenty dat
    - komponenty grafického výstupu (prostor atributů)
    - objekty, 3D plochy - opět zoom, otáčení, výběr objektu, perspective walls
    - změny struktury vizalizace - př. skrytí komponent

### Fish eye
One of the popular transformations of this type is given by the formula:
$r_{new} = s*log(1+d(r_{old}))$
where
$s=\frac{r_1}{log(1+d*r_1)}$
This formula ensures that the radius of the points located at the edge of the magnifying glass is maintained at its original value. Now let's look at the pseudocode of the whole algorithm:
1. Clean the output image.
2. For each pixel of the input image:
	1. Calculate the corresponding polar coordinates.
	2. If the radius is smaller than the radius of the magnifying glass:
		1. Calculate the new radius rnew.
		2. We get the color of this place from the original image.
		3. Set this color as the pixel color in the output image.
	3. Otherwise, set the resulting pixel of the image to the same value as in theoriginal image
![Fisheye|300](../obrazky/8_vizualizace/fisheye.png)
Possible problems with the transformation:
- pixel overlaps (minor problem)
- pixel holes (bigger problem - solved using interpolation)
### Perspective walls
- typical example of remapping in 3D
- Used for example for large number of documents and data
- Shows one panel of the view and other panels are oriented such that they disappear in a distance
![[Pasted image 20240601142559.png|400]]
## Základní principy vizualizace

![Pipeline](../obrazky/8_vizualizace/pipeline.png)

- pipeline
    - zdroj dat $\to$ tabulky $\to$ vizuální abstrakce $\to$ pohledy, kdekoliv v procesu může uživaltel interagovat (na bodech i šipkách), šipky "many-to-many"
- grafické symboly (šipky, labely, značky)
    - semiologie
    - často využíváme předchozí znalosti a zkušenosti - př. dopravní značky
    - detekce vzorů a vztahů v "obrazu"
        - každý vzor v obrazu by měl odpovídat vzoru v datech, jinak jde o rušivý prvek
        - pokud na obrázku vidíme nějaké uspořádání, mělo by být i v datech, jinak ruší vjem
    - když vnímáme složtější obraz, tak nejprve prvky, pak vztahy (kognitivně)
        - prvky vnímáme hned, podvědomě
        - některé vlastnosti prvků vnímáme rychleji - barva vs tvar
- dimezionalita ve 2D grafice
    - můžeme zobrazit víc než dvě dimenze (velikost a barva symbolů, ...), ale musíme myslet na to, že prvky mají být rozlišitelné
- formalizace
    - Sémiologie Graphique, 1967 Jacques Bertin
        - separace obsahu (informací, které chceme zakódovat) a přenosového média (použitý grafický systém)
        - grafický slovník:
            - značky - body, přímky, plochy
            - pozice - dvě dimenze, xy
            - další - velikost, textura, barva, ...

## Geodata

- klima, telefonní hovory, platby kartou, ... cokoliv, kde nás zajímá, kde na světě se to dělo, zobrazujeme na mapu
- mapa - zjednodušené zobrazení světa, body, čáry, plochy
- co chceme zobrazovat 
    - body, čáry (linie), plochy, povrch (2.5D)
- typy map podle vlastností zobrazených dat
    - symbolické (nominální bodová data)
    - bodové (ordinální bodová data)
        - pozor na moc husté a překrývající se body
    - land-use (nominální plošná data)
    - colorpleth maps - nějaký fenomén v dané oblasti mapujeme na barvu oblasti
        - pokud fenomén tvoří jiné oblasti, než je rozdělení mapy, tak je lepší použít asymetrické mapy (prostě ne podle těch regionů)
    - liniový diagram (nominální nebo ordinální data čar)
    - isolines, tedy vrstevnice (ordinální plošná data)
    - mapy povrchu (ordinální objemová data)
- kartogram - deformace ploch podle dat (př. populace)
- explorativní vizualizace - zásadní prvek jsou interakce
- mapové projekce
    - zachovávají různé vlastnosti
        - conformal - zachovává lokální úhly/tvary, nezachovává plochu
        - equivalent - oblast zabírá stejnou plochu na mapě a projekci, mění tvary a úhly
        - equidistant - zachovává vzdálenosti od bodu/čáry
        - gnomonic - zachovává nejkratší vzdálenost mezi body
        - azimuthal - zachovává směr od centrálního bodu
    - podle typu povrchu
        - cylindrická - omotaný papír kolem koule, je conformální
        - pseudocyrindlická - poledníky do oblouku, hlavní poledník a rovník rovně
        - planární - připlácneme papír na kouli
        - kónická - dáme na kouli papírovou čepici a pak rozstříhneme
    - další
        - equirectangular
        - equal-area conic
        - cosinusoidal
        - ... 
- sítě - problémy: překrývající se hrany
- labeling map - je to složitější, než to vypadá
- kartogramy
    - viz kolečka níže, vybarvení příslušně škálované části (pouze vnitřek), zvětšování částí (a tím deformace celku)
    - generování
        - manuálně náročné, hodně se to studuje
    - efektivní = rychle předá informaci, rychle je vidět i vztah s původní mapou

![Kartogram](../obrazky/8_vizualizace/kartogram.png)
    

## Textové dokumenty

- textu existuje hrozně moc - co s ním chceme dělat?
    - hledat slova a fráze
    - hledat témata
    - hledat vzorce ve strukturovaných textech nebo složkách
    - vztahy mezi odstavce/částmi/dokumenty
- textová reprezentace
    - lexikální - množiny písmen transformujeme na tokeny
        - extrakce nejčastěji pomocí konečných automatů a regulárních výrazů
    - syntax - k tokenům přidáme značky - pozice ve větě, zda je to sloveso, ...
    - sémantika - extrahuje význam a vztahy
- vector space model (VSM)
    - dokumenty jako vektory, každá dimenze vektoru přísluší nějakému termínu (slovu, tokenu) - je li termín v dokumentu, je hodnota tímhle směrem nenulová
    - váhy termínů - term freq. inverse document freq. (tf-idf)
        - počet výskytů termínu v dokumentu tf(w), počet dokumentů, kde se termín vyskytuje df(w), počet dokumentů n
        - $tfidf(w) = tf(w) * \log(N/idf(w))$
    - Zipf's law - distribuce slow v korpusu 
        - frekvence slova v dokumentu je inverzně úměrná jeho ranku ve frekvenční tabulce
- vizualizace dokumentů 
    - tag cloud - velikost slov odpovídá frekvenci
    - word tree - kořen slovo, větve kontexty
    - text arc - tag cloud + hrany k dokumentům na obvodu
    - arc diagram - spojuje stejné subsekvence
    - document fingerprint - pixel prezentuje blok textu, barva př. délku vět
    - slef-organizing map 
    - themescapes - 3D krajina popisující korpus, kopce tam, kde je víc podobných dokumentů
    - document cards - obrázky a klíčová slova
    - můžeme vizualizovat kód - každý řádek obarvíme třeba podle toho, kdy byl naposled upravený, jak často se volá nebo tak
    - vizualizace výsledků hleání - obdélník odpopídá délce textu, barva pixelů odpovídá výskytu hledaných termínů (každý termín v jednom řádku)
    - theme river
    - síť dokumentů, u každého vrcholy s termíny