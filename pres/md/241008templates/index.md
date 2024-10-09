<div id="markdown-custom-center">HiCollege - C++ Templates</div>
<!-- .slide: class="center" style="font-size:80%; padding-left:20%; margin-top: -1%; width:80%;" data-background-transition="slide" data-background-image="images/ppt-arno-1.png" data-state="hide-footer" -->

# HiCollege <br/> C++ Templates
## Arno Lepisk
### <code style="background: inherit;">arno.lepisk@hiq.se</code>
### <span class="twitter"><code style="background: inherit;">@arno_l</code></span> &nbsp; <span class="github"><code style="background: inherit;">f00ale</code></span>
### 2024-10-08

---

# Introduktion
* Templates är C++ sätt att gör metaprogrammering
  * "programmera programmering"
  * generisk programmering
* exempel i compilerexplorer (aka godbolt)

--
# Varför templates
* Förenklar typriktig programmering
* Möjliggör återanvändning av kod
* Gör att man nästintill kan skapa egna språk ovanpå C++
* C++ kan i viss mån behandlas som duck-typing

---
# Befintliga templates
* Standardbiblioteket med STL innehåller flertalet templates som kan användas
* Exempel
  * `std::vector`
  * `std::array`
  * `std::sort`
  * `std::tuple`
* [https://godbolt.org/z/WoYbY3Tcj]

--
# Befintliga templates (forts)
* Templates har ofta default-argument, som kan överridas
* Templates kan även ha specialiseringar, som kan ha andra (oväntade) beteenden...
* [https://godbolt.org/z/q69se7dEn]

---
# Skriva egna templates
* Generell syntax för templates
```c++
template <typename T> 
struct MyTemplate 
{ ... };
```
* Tidigare var det vanligt med `class` istället för `typename`, men `typename` är i mitt tycke tydligare trots att det är längre
* [https://godbolt.org/z/T8b57es44]

--
# Beroenden
* Vissa saker kan kompilatorn inte lista ut
* Ibland måste man hjälpa kompilatorn med att lista ut saker
* [https://godbolt.org/z/M4rerWKh5]

--
# Specialiseringar
* syntax för en komplett specialiering (vi kommer till icke-kompletta senare)

```c++
template<typename T> 
struct MyTemplate { ... };

template <> 
struct MyTemplate<ImSpecial> { ... };
```
* fungerar även för funktioner, men har då vissa begränsningar. Enklast att 
  stoppa funktioner som statiska funktioner i en klass i de fallen. Man kan
  då ha en hjälpfunktion för att hitta rätt.

* [https://godbolt.org/z/jz1Ej5hEx]

--
# "dolda templates"
* Ett lambda med auto-argument är i praktiken ett funktions-template 
  * (för att komplicera det ytterligare kan man ha template-argument till lambdas i senare C++-varianter)
* [https://godbolt.org/z/EzY6KvjYc]

-- 
# templates som templateargument
* Ibland vara explicit med att ett templateargument ska vara ett template
* [https://godbolt.org/z/cn17cE5Ma]
* (här fallet där man länge var tvungen att använda `class` istället för `typename`)

--
# statiska variabler i templateiserade
* Varje template instansiering får sitt eget context
* [https://godbolt.org/z/b4rMMTx17]
* 
--
# debugga templates
* svårt.
* Man kan ta länkaren till hjälp (eller snarare felmeddelanden från länkaren)
* cppinsights
* [https://godbolt.org/z/dr85Gb3hY]
---
# Övning 1
* Skapa en template-klass som har två medlemmar av en given typ.
  * Kalla template-klassen MyTemplate
* Skapa en metod som skriver ut summan av dem.
  * kalla metoden print
* Extra: om typen är int - dubbla det utskrivna värdet
* [https://godbolt.org/z/rsn5K1Es1]


--
# Lösningsförslag
* [https://godbolt.org/z/Tjjobjhh9]

---
# Typ-hantering
* `&&` (forwarding reference/universal reference, rvalue reference)
* `forward` - kan vidarebefodra argument med rätt typ. Kan vara viktigt när man kedjar anrop.
* [https://godbolt.org/z/bbjGrn43r]
* type_traits - kan svara på frågor och "modifiera" typer. 
  * `std::remove_cv_t` och `std::remove_reference_t`
  * [https://godbolt.org/z/rvPT36oM3]
  * [https://godbolt.org/z/8hE7531G3] 
--
# Typer (forts)
* `decltype`/`std::declval` - ger typen av en deklaration (även returvärde från en funktion)
  respektive ett fiktivt objekt av en typ att evaluera i context.
  * [https://godbolt.org/z/ds6o3YWsM]
  * [https://godbolt.org/z/YjMx88TMj]
---
# SFINAE
* Substituition Failure Is Not An Error
* Betyder i korthet att vissa fel (substitueringsfel) inte är kompileringsfel, utan de fallen ignorerars bara
* nyttig helper: `std::void_t`
* [https://godbolt.org/z/37hrej7cM]
* [https://godbolt.org/z/dfdjar18c]
* [https://godbolt.org/z/3qx7T3sY8]
---
# Övning 2
* Givet två containers som har metoder `IsEmpty` respektive `empty` - skriv
  en funktion `general_empty` som anropar rätt och returnerar
* tips: titta på SFINAE, `std::declval` och `decltype`
* [https://godbolt.org/z/3vz5GaWaf]

--
# Lösningsförslag 2
* [https://godbolt.org/z/8P5P8dovd]
---
# CRTP
* Curiously Recurring Template Pattern
* En teknik för att "vända på arv", man "injekterar" beteende in i en basklass
  och får till polyformism utan virtuella funktioner. (det blir dock en slags virualism)
* Man kan även få "polyformism" för statiska funktioner
* [https://godbolt.org/z/j8bqWv4zG]
---
# Variadiska templates och typlistor
* Ett template med godtyckligt antal templateparametrar
* [https://godbolt.org/z/vW41nqnMc]
* [https://godbolt.org/z/vW3ch4Edc]

--
# Typlistor
* [https://godbolt.org/z/5EPM39jMP]
* [https://godbolt.org/z/qzzY6qae6]
* [https://godbolt.org/z/6xj8KKnhE]

---
# Nackdelar med templates
* Kompileringstider
  * mycket kod måste läggas i header-filer
* Kodduplicering i binärer
  * Debug-info kan bli stort
* Ofta svårläst kod
* Svårt att felsöka

