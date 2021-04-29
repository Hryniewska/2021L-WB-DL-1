### Grupa: Kurowski Mróz Podsiad

## Praca domowa nr 3 - Preprocessing

### 1. Preprocessing w artykule 

Autorzy artykułu wykorzystują dwie metody preprocessingu:

* Wyrównanie histogramu
* Usunięcie adnotacji 

Pierwsza z metod polega na rozciągnięciu wartości pikseli będących reprezentacją zdjęcia w taki sposób, że zostaje zwiększony jego kontrast. Po przedstawieniu ich w formie histogramu proces rozszerza go w ten sposób, że wartości są bardziej równomiernie rozłożone na całej dostępnej skali jasności pikseli.

Natomiast usunięcie adnotacji to wykorzystanie biblioteki OpenCV do usunięcia najbardziej odstających od reszty obrazu artefaktów (takich jak litery L, R oznaczające strony klatki piersiowej widoczne na zdjęciu). Jest to wykonywane dla każdego obrazu dwa razy (sprawdzanie prawej i lewej strony), po uprzednim usunięciu szumu z wykorzystaniem funkcji anisotropic_diffusion z biblioteki medpy.

Niestety kod autorów w surowej formie był niemożliwy do uruchomienia. Brakowało importów potrzebnych bibliotek oraz sam główny kod nie został dokończony. Dlatego zmodyfikowaliśmy kod autorów dodając swoje własne fragmenty tam gdzie było to potrzebne.

Przykładowy preprocessing obrazu zamieszczamy w notebooku *image_preprocessing.ipynb*.

Oryginalny kod autorów znajduje się w pliku *CXR_preprocessing.py*.

Fragment artykułu:

> "Since radiographs usually have dark edges, images with such distinctly darker regions impact the classification. We perform global contrast enhancement, edge enhancement, and noise elimination on entire CXR images with histogram equal ization and unsharp masking edge enhancement."

### 2. Nasze ulepszenia

Pierwszą z metod preprocessingu, którą sprawdziliśmy jest Szybka Transformata Fouriera (FFT). Przetransofrmowane zostały zdjęcia wybrane podczas undersamplingu a następnie wytrenowaliśmy sieć VGG na ich podstawie. Plik transformujący zdjęcia to RUS_with_fft.ipynb. Uzyskana macierz pomyłek:

<p align="center">
<img src="https://i.imgur.com/HdiSBUR.png">
</p>

Drugą metodą było przepuszczenie zdjęć przez filtr SHARPEN dostępny z paczki PILLOW. Ułatwia on odróżnianie różnych części obrazu od siebie. Zwiększa on również jasność jaśniejszych pikseli. Plik transformujący zdjęcia to RUS_with_sh.ipynb.
Uzyskana macierz pomyłek:

<p align="center">
<img src="https://i.imgur.com/LIfShEf.png">
</p>

Trzecią metodą będzie nasza własna modyfikacja kodu autorów. Jako, że nie dało się go uruchomić, wprowadziliśmy dużo zmian i usunęliśmy niepotrzebne procedury. Finalnie nasz preprocessing składa się z następujących kroków: 
 - zmiana rozmiaru zdjęcia z antialiasingiem, 
 - rozciągnięcie histogramu (zwiększenie kontrastu), 
 - redukcja zaszumienia przy pomocy funkcji anisotropic_diffusion, 
 - przekonwertowanie z formatu RGB do greyscale dla szybszego działania, 
 - stoworzenie maski za pomocą funkcji threshold z biblioteki OpenCV (wykrycie najjaśniejszych miejsc na zdjęciu),
 - modyfikacja (pogrubienie) maski za pomocą funkcji dilate z OpenCV (żeby możliwe było wykrycie cienkich linii),
 - usunięcie adnotacji z obrazu za pomocą funkcji inpaint z OpenCV i wykorzystaniem stworzonej już maski.

Zrezygnowaliśmy natomiast z podziału obrazów na dwie części odpowiadające prawej i lewej stronie.

Przed: 
<p align="center">
<img src="https://i.imgur.com/QBRZ3yR.png" height="400px">
</p>

Po: 
<p align="center">
<img src="https://i.imgur.com/h6Psomm.png" height="400px">
</p>

### 3. Preprocessing w OCR

Preprocessing w OCR (optycznym rozpoznawaniu znaków) jest często stosowany żeby poprawić precyzję. Chcemy żeby obrazy, na których wykonywany jest OCR miały jak najbardziej czytelne znaki (wysoką ostrość, duży kontrast, dobre wypoziomowanie, małe szumy). Poniżej techniki które wykorzystywane są do polepszenia tak zdefiniowanej jakości zdjęć:

* **Binaryzacja**, czyli konwersja kolorowego zdjęcia na takie, które zawiera tylko białe i czarne piksele. Określony musi być odpowiedni *threshold*, według którego odpowiednio klasyfikowane są piksele. *Threshold* może być ustalony jeden dla całego obrazu albo może mieć różne wartości dla różnych części zdjęcia (*adaptive thresholding*).
<p align="center">
<img src="https://miro.medium.com/max/2400/1*KTMCWiv4WZRlIpZSs5kupw.jpeg">
</p>

* **Korekcja nachylenia**, czyli przekrzywienie obrazu w celu nadania mu właściwego formatu i kształtu - tekst powinien pojawiać się poziomo i nie powinien być pochylony pod żadnym kątem. Może to być wykonane w na przykład taki sposób:
    - rzutujemy binaryzowany obraz poziomo (czyli bierzemy sumę pikseli wzdłuż wierszy macierzy obrazu), aby uzyskać histogram pikseli wzdłuż wysokości obrazu
    - obracamy obraz w małym przedziale kątów i obliczamy różnicę między wierzchołkami histogramu. Kąt, przy którym mamy maksymalną różnicę między wierzchołkami to nasz kąt nachylenia.
    - po znalezieniu kąta nachylenia obracamy nasz obraz o kąt przeciwny do kąta nachylenia i otrzymujemy dobrze wypoziomowany obraz.
<p align="center">
<img src="https://miro.medium.com/max/610/1*b76nvNcSUNwMnBdm1rd7WQ.gif">
</p>

* **Odchudzanie i szkieletowanie**, których to używa się w przypadku gdy badamy tekst pisany ręcznie. Procesy te pomagają nam uzyskać jednolitą grubość kresek, co polepsza dokładność wyników OCR.

* **Usuwanie szumów**, którego głównym celem jest wygładzenie obrazu poprzez usunięcie małych kropek, które mają większą intensywność niż reszta obrazu. Usuwanie szumów może być wykonywane zarówno dla obrazów kolorowych jak i binarnych.

Poniżej przykład preprocessingu wykonanego na 3 próbkach odręcznie pisanego tekstu.

<p align="center">
<img src="https://miro.medium.com/max/700/1*Yajazz-a5PwbFOYS9w7nlg.png">
</p>

### 4. Wykrywanie liter, artefaktów i image inpainting

Jak można się domyślić, problem pozbywania się liter z obrazków sprowadza się do dwóch mniejszych problemów: wykrycia tekstu oraz wypełnienia go czymś “sensownym” dla ludzkiego oka.

**Wykrywanie liter na zdjęciach**: 

Proces ten polega na rozpoznaniu symboli literowych (zależnie od języka niekoniecznie to są litery). Przeważnie symbole te cechują się zwiększoną gęstością kantów i/lub większym kontrastem okolicznego miejsca na obrazie.

*Porównanie technik*:

- Oparta na Laplasjanie: opiera się na przejściu wpierw maską o sumie 0 i większej wartości w środku przez obraz w celu znalezienia miejsc o największym kontraście. Potem, szukając największego gradientu, wyszukujemy piksele  tekstowe, które łączymy w bloki. 
<p align="center">
<img src="https://iq.opengenus.org/content/images/2020/05/download-3.png" width="500px">
</p>

- Oparta na kantach: Opiera się na wyszukiwaniu kantów charakterystycznych dla bloków tekstowych. W szczególności, znajdowane są wierzchołki bloku tekstowego.
<p align="center">
<img src="https://i.imgur.com/LUjDqVK.png">
</p>

- Oparta na DCT: DCT (discrete cosine transform) jest metodą podobną do discrete fourier transform, ale rezultatem są tylko liczby rzeczywiste. 
<p align="center">
<img src="https://i.imgur.com/HNNETtl.png">
</p>

**Wykrywanie artefaktów na zdjęciach**:

Proces ten polega na wykrywaniu obiektów na zdjęciach, które są niepożądane - przykładowo, elementy ubioru, czy fałdy skórne na zdjęciu rentgenowskim mogłyby być mylnie odczytane jako choroba. Innym przykładem są nieodwracalne zmiany powstałe podczas dekonwolucji. W niektórych przypadkach regiony te cechują się np. Większym kontrastem, czy jasnością pikseli.
Jedną z metod wykrywania artefaktów jest wpierw zastosowanie filtrów Gabor. Filtr ten uzyskuje lokalną informację przestrzenną i częstotliwościową, zaś później odpowiednia sieć na podstawie tej informacji usuwa artefakty.

**Image inpainting** 

Jest to proces rekonstrukcji brakujących części obrazu w taki sposób, że obserwatorzy nie są w stanie stwierdzić, że te regiony zostały poddane rekonstrukcji. Technika ta jest często używana do usuwania niepożądanych obiektów z obrazu lub do przywracania uszkodzonych fragmentów starych zdjęć.

Początkowo wymagała ona ręcznego wykonania pracy przez człowieka. Obecnie jednak istnieje wiele metod automatycznego wypełniania. Oprócz obrazu, większość z tych metod wymaga również jako danych wejściowych maski pokazującej regiony wymagających wypełnienia. Regiony te mogą być podane przez człowieka lub automatycznie wykryte.

*Porównanie technik*:

**PDE (Partial Differential Equation)**:
- Wykorzystuje podejście oparte na izofotach (krzywa na oświetlonej powierzchni, która łączy punkty o jednakowej jasności, w tym przypadku linie proste o równych wartościach w skali szarości) i zachowuje wszystkie informacje o strukturze
- Działa na poziomie pikseli, daje efekt rozmycia dla większych obszarów i zajmuje dużo czasu. Brzegi są przedłużane w prostej linii, dlatego ten algorytm nie jest odpowiedni dla krawędzi zakrzywionych
<p align="center">
<img src="https://i.imgur.com/C35D9rQ.png">
</p>

**Texture Synthesis**
- Dobrze radzi sobie z przybliżaniem tekstur
- Trudność w obróbce naturalnych obrazów i słaba rozdzielczość przy usuwaniu dużych obiektów
<p align="center">
<img src="https://i.imgur.com/BlacVEb.png">
</p>

**Exemplar**
- W tej technice wybierane są najlepiej pasujące fragmenty ze znanych regionów zdjęcia za pomocą określonych metryk, a następnie wstawiane są one w wypełniany obszar
- Wydajny dla większych obszarów docelowych
- Dłuższy czas obliczania
<p align="center">
<img src="https://i.imgur.com/KZ694B8.png">
</p>

**Total Variation**
- Najlepiej nadaje się, gdy obszar do wypełnienia jest cienki (np. tekst)
- Doskonale nadaje się do usuwania szumu ze zdjęć
- Radzi sobie tylko z wypełnianiem małych obszarów obrazu
<p align="center">
<img src="https://i.imgur.com/DzsBVI2.png">
</p>

**Convolution**
- Daje dobre rezultaty: bez rozmycia, szybkie, iteracyjne, proste do wdrożenia
- Przy usuwaniu dużych obiektów z naturalnych obrazów uzyskuje się niewyraźne krawędzie
<p align="center">
<img src="https://i.imgur.com/r83vLD1.png">
</p>

**Hybrid**
- Hybryda PDE i syntezy tekstur
- Przywrócona zostaje gładkość, a struktura i tekstura obrazu zostają zachowane
- Jeśli wypełniany obszar jest duży i niewłaściwie dobrany, uzyskuje się blokowaty obraz
<p align="center">
<img src="https://i.imgur.com/wzy9mao.png">
</p>


