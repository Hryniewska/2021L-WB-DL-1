# Rozwój modelu poprzez dodanie augmentacji danych oraz transfer learningu

## 1. Augmentacja danych

_Zaugmentuj dane przy pomocy Generative Adversarial Network (GAN). Podejścia dowolne. Można zaimplementować samemu. Można poszukać, czy jest może jakaś odpowiednia w Internecie._

### 1.1 Przegląd literatury
#### 1.1.1 [Data augmentation using generative adversarial networks (CycleGAN) to improve generalizability in CT segmentation tasks](https://www.nature.com/articles/s41598-019-52737-x) (V. Sandfort, K. Yan, P. Pickhardt & R. Summers, 2019)

Autorzy artykułu badają jak sztucznie wygenerowane dane przy pomocy CycleGAN wpływają na wyniki segmentacji trenowanej sieci. Transformacja przeprowadzana przez CycleGAN polegała na obniżeniu kontrastu zdjęcia CT. Trenowanie sieci zostało przeprowadzone na zbiorze oryginalnym, a także na zbiorze będącym połączeniem zbioru oryginalnego ze zbiorem syntetycznym składającym się ze zdjęć o obniżonym kontraście. Wyniki sieci zostały również porównane z wynikami sieci trenowanej na powiększanych zbiorach danych z użyciem takich metod jak standard augmentation oraz histogram equalization augmentation. Okzało się, że CycleGAN w wielu przypadkach w zależności od organu wewnętrznego sprawuje się najlepiej.

#### 1.1.2 [Unpaired Image-to-Image Translationusing Cycle-Consistent Adversarial Networks](https://arxiv.org/pdf/1703.10593.pdf) (J. Zhu, T. Park, P. Isola, A. Efros, 2017) 

Powyższy artykuł opublikowany w 2017 r. prezentuje rozwiązanie CycleGAN. Jest w nim zawarte wyjaśnienie jak działa CycleGAN. Kod napisany przez autorów został napisany pod bibliotekę PyTorch, jednak jest dużo powszechnie dostępnych implementacji CycleGAN dostosowanych do biblioteki Tensorflow.
Na oficjalnej stronie Tensorflow dostępny jest tutorial wprowadzający do korzystania z CycleGAN w ramach dogłębnego zrozumienia jego działania strona odsyła nas do wyżej wymienionego artykułu. (https://www.tensorflow.org/tutorials/generative/cyclegan)

#### 1.1.3 Podsumowanie

Augmentacja zdjęć CT klatki piersiowej została już częściowo zgłębiona. W pierwszej z przytoczonych przeze mnie prac autorzy wykorzystują powiększony zbiór danych do segmentacji nerek, wątroby oraz śledziony. Moim celem będzie sprawdzenie czy zastosowana metoda obniżania kontrastu zdjęć i mieszania ich ze zdjęciami oryginalnymi pomoże osiągnąć dokładniejsze wyniki dla segmentacji płuc w sieci BCDU. 
Narzędziem którego zamierzam użyć do augmentacji zbioru zdjęć będzie CycleGAN.

## 2. Transfer learning - auxiliary task

_Wymyśl dodatkowe zadanie (auxiliary task), które będzie dobrze nadawało się do Twojego problemu. Wykonaj uczenie wstępne za pomocą dodatkowego zadania._

### 2.1 Przegląd literatury
W ramach przygotowań do zaaplikowania dodatkowego zadania do modelu sieci neuronowej rozwiązującej zadanie segmentacji obrazów tomografii komputerowej płuc, dokonałem przeglądu istniejącej literatury naukowej, w celu poszukiwania wiedzy i inspiracji w jaki sposób najlepiej podejść do zagadnienia. Poniżej znajduje się kilka pozycji, które mogą się okazać pomocne w tym przypadku.

#### 2.1.1 [Multitask learning: teach your AI more to make it better](https://towardsdatascience.com/multitask-learning-teach-your-ai-more-to-make-it-better-dde116c2cd40) (Honchar A., 2018)

Artykuł został opublikowany na znanym i cenionym portalu towardsdatascience.com. Prezentuje uzasadnienie użycia dodatkowego zadania w procesie rozwoju modelu sieci neuronowej korzystając z różnych intuicji oraz przedstawia kilka praktycznych przypadków użycia techniki wraz z załączonym kodem napisanym w Pythonie z pomocą biblioteki Keras. Czytelnik ma okazje niemal namacalnie zauważyć jakie korzyści mogą płynąć z użycia dodatkowego zadania. Autor podkreśla, że technika może być uzyta praktycznie w przypadku każdego modelu sieci neuronowej wyliczając rozmaite pomysły jak osiągnąć lepsze wyniki na przykład rozpoznawania emocji na podstawie zdjęć twarzy czy przewidywania przyszłych indeksów giełdowych. Dla ciekawych historii i inspiracji pomysłu podany jest także krótki rys historyczny.

#### 2.1.2 [An Overview of Multi-Task Learning in Deep Neural Networks](https://arxiv.org/pdf/1706.05098.pdf) (Ruder S., 2017)

Opublikowana w 2017r. praca badacza z uniwersytetu w Dublinie stanowi przekrój ówcześnie używanych metod multitasku w modelach uczenia maszynowego. Artykuł składa się z wstępu - motywacji używania techniki, rozróżnienia dwóch wariantów: "Hard parameter sharing" oraz "Soft parameter sharing", wyjaśnienia skuteczności techniki, przykładu użycia w płytkich oraz głębokich modelach oraz propozycje dodatkowych zadań i konkluzje. Wśród nowych propozycji używania MTL (Multi-Task Learning) w sieciach neuronowych można wyróżnić m.in.:
* Głęboko powiązane sieci - dotrenowanie niektórych ostatnich warstw konwolucyjnych uniwersalnego modelu (np. ImageNet),
* Pełne dzielenie się zmiennymi - procedura dzielenia modelu sieci neuronowej na niezależne komponenty wraz z przebiegiem procesu trenowania,
* Połączone modele wielozadaniowe

Opisane w artykule techniki stanowią raczej wskazówki dla zaawansowanych autorów sieci neuronowych. Ponadto, Ruder przedstawia kilka propozycji zadań dodatkowych, takich jak np. "Wskazówki" - predykcja cech istotnych w modelu.

#### 2.1.3  [MULTI-TASK LEARNING FOR THE SEGMENTATION OF THORACIC ORGANS AT RISK IN CT IMAGES](http://ceur-ws.org/Vol-2349/SegTHOR2019_paper_2.pdf) (Tao H. et al., 2019)

Autorzy powyższego artykułu zmierzyli się z tematyką podobną do pola działania opracowywanego przez nas modelu - segmentacją obrazów tomografii komputerowej płuc. Badacze za pomocą dodatkowego zadania nauczyli sieć identyfikować występowanie poszczególnych organów - przełyku, serca, tchawicy i aorty, co może być istotne w ich analizie. Zostało to dokonane przy pomocy Encodera w kształcie sieci U. Proponowany kształt modelu może być silną inspiracją w rozwijaniu transfer learningu w naszym projekcie.

#### 2.1.4 Podsumowanie

Świat uczenia maszynowego rozwija się z wysoką prędkością. Z pewnością można uzyskać dostęp do większej liczby opracowań na temat wykorzystywania dodatkowych zadań w sieciach neuronowych, jednak powyższe pozycje powinny być wystarczające do implementacji pierwszego takiego rozwiązania. Prezentowane prace zainspirowały do stworzenia zadania identyfikującego poszczególne organy (do czego niezbędne byłoby stworzenie odpowiednich etykiet wykorzystując wiedzę dziedzinową) czy też regresji liczby niezerowych pikseli na podstawie obrazu tomografii.

## 3. Transfer learning - unsupervised pretraining

_Przeprowadź nienadzorowane uczenie wstępne modelu (unsupervised pretraining)._

### 3.1. Przegląd literatury


#### 3.1.1 [Pre-Training CNNs Using Convolutional Autoencoders](https://www.ni.tu-berlin.de/fileadmin/fg215/teaching/nnproject/cnn_pre_trainin_paper.pdf) (Maximilian Kohlbrenner et al.)

W tej publikacji opisane jest zastosowanie autoenkoderów do inicjalizacji wag modelu. Autorzy twierdzą, że dzięki tej metodzie accuracy ich modelu podniosło się z poziomu 0.7 do 0.736 tylko dzięki zastosowaniu uczenia wstępnego, co stanowi znaczną poprawę. W artykule tym bardzo dokładnie opisana jest architektura CAE (Convolutional Auto Encoders), które to stosowane są do wstępnego uczenia, a następnie ich wagi przepisywane są do warstw głównego modelu. Dzięki opisowi konkretnych parametrów warstw, będziemy mogli spróbować zastosować dokładnie tę samą architekturę w naszym projekcie i sprawdzić, czy również pozwoli na poprawę wyników.

  #### 3.1.2 [Unsupervised Pre-training Across Image Domains Improves Lung Tissue Classification](https://link.springer.com/chapter/10.1007/978-3-319-13972-2_8) (Thomas Schlegl et al. 2014)

Autorzy tej publikacji zastosowali Unsupervised pretraining jako sposób na poradzenie sobie z małą ilością danych treningowych. Tylko część obserwacji ze zbioru testowego była opisana etykietą, a co za tym idzie tylko ta część mogła być użyta do uczenia nadzorowanego. Naukowcy mieli jednak dużo więcej rekordów nieoznakowanych i za ich pomocą przeprowadzili trenowanie wstępne. Użyli oni CRBM (Convolutional Restricted Boltzmann Machine) jako model, który zainicjalizuje wagi poszczególnych warstw. Mamy nadzieję przetestować również to podejście w zastosowaniu do naszego projektu.


### 3.1.3 Podsumowanie

Unsupervised pretraining jest używany razem z modelami konwolucyjnymi dość często. Pozwala na użycie nieopisanych danych jako początek uczenia modelu, co jest bardzo przydatne, ponieważ opisane dane medyczne są zazwyczaj trudno dostępne. Mamy nadzieję, że opisane prace pozwolą nam zaimplementować tę funkcjonalność do naszego modelu.

