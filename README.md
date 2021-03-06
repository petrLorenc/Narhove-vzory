# Navrhove vzory
Budu se zde snažit, svými slovy, popsat co znamenají návrhové vzory, co znamená **OOP** a "obecně dobré" programovací přístupy. Nekdy budu psat bez hacku a carek, protoze to je pro me pohodlnejsi a stejnak mam v planu cely tento clanek prepsat do anglictiny.

Cast 1
Budu se snazit udrzovat jednotnou strukturu a vsechny zdroje uvadet na konci (a vsechny pohromade). Jako hlavni vychozi bod jsem zvolil knizku **Navrhove vzory od Rudolfa Pecinovskeho** a jako rychly prehled jsem zvolil stranky www.tutorialspoint.com/design_pattern/index.htm Dale mi prijdou stojici za precteni stranky http://www.algoritmy.net/ , ktere vychazeji z jiz vyse zminene knizky. Vsechny kody (u jasnych myslenek nebudu uvadet) se budu snazit testovat v **Jave** (jako momentalni Androidar se pokusim priblizit nejake i k Androidu).

# Zaklad
Mezi zakladni zasady objektove orientovaneho programovani bych zvolil tyto:
* **Programovani proti rozhrani** a uprednostnovat rozhrani pred tridami v dedeni
  * Jedna z vyhod uprednostnovani rozhrani (interface) pri dedeni je ze muzete dedit od vice rozhrani
  * Vazba mezi tridami (pri pouziti treba abstract class) pri dedeni je velmi silna (vetsinou nechceme)
  * Abstraktni tridy se vyplati pokud vime ze se bude API menit - neni nutnost velkych zmen (pridani metody k rozhrani nas ale nuti implementovat ji vsude kde se pouziva toto rozhrani)
* Soudrznost (**Cohesion**) tridy
  * Kazda trida by mela soustredovat na jednu konkretni vec -> snadny testing
* Nizka vzajemna provazanost (**Coupling**)
  * Pokud by jsem odstranili jednu tridu tak by nemelo byt nemozne ziskat zpatky funkcni kod
* Neopakovat stejny kod - **DRY** (Dont repeat yourself)
  * Zakladni princip, nedodrzeni vede k mnozstvi chyb z nepozornosti
* Neukazovat ostatnim co nepotrebuji - **zapouzdreni**
  * Nemelo by se stavat ze ostatni objekty dostanou informace ktere nutne nepotrebuji
  * Muze vest k ne/zadoucim zmenam vnitrnich stavu promnenych a tudiz k chybam

# Simple factory method - Jednoducha tovarni funkce
Jedna se o **statickou metodu** kterou budeme volat na tride k navraceni instance teto tridy. Mezi hlavni vyhody bych zaradil ze se nam nutne **nemusi vratit novy objekt** (na rozdil od konstruktoru, ktery ho vzdy vytvori) a neni nutnost volat metody predka hned jako prvni vec. **Muze dokonce vratit instanci nektereho potomka**. V Android prostredi se casto vola metoda **newInstance**(...) ktera svym zpusobem je SFM (z volby jmena by melo byt jasne co funkce vraci - novy objek, singleton ...).

#  Immutable object
* Snazit se pouzivat nemenne objekty (immutable object)
  * Snadnejsi kontrola stavu objektu
  * **Thread-safe**
  * Neni nutne pouzivat defenzivni ziskavani objektu (tj. vrati kopii promenne - aby se nemenil vnitrni stav objektu odkud promnenou bereme)
  * pouziti == je na testovani referenci jestli ukazuji na stejne misto, Object.equal(Object) se vaze k metode equals a hashCode
  * pokud se pokusim zmenit immutable objekt vetsinou dostanu novy a neprojde pres == test (String je immutable)
```Java
String s1 = "a"; // s1 point to adress 0x000001 where is "a"
s2 = s1; // s2 point to adress 0x000001 where is "a"
s1 = s1 + "b"; // it take whatever s1 is pointing to and create new object at adress 0x000002 and add "b"
System.out.println(s1 == s2); // prints false
```
```Java
StringBuffer sb = new StringBuffer("a"); // sb point to adress 0x000001 where is "a"
StringBuffer sb2 = sb; // sb2 point to adress 0x000001 where is "a"
sb.append("b"); // doenst create new object and just add "b" to "a" to get "ab"
System.out.println(sb == sb2); // prints true
```
Difference between mutable and imutable http://stackoverflow.com/a/30835852/1551954 (examples from there)
# Prepravka - Crate - Messenger
* **interni** - slouzi pro uchovani hodnot (vnitrnich stavu) nejake tridy - ukladani informaci o casovem prubehu udalosti o dni (ostatni o teto tride nemusi nic vedet)
* **externi** - slouzi k vraceni vice nez jedne navratove hodnoty (volajici a cilova trida ma prepravku videt)
* jsou definovany jednoucelove a nemeli by se "recyklovat" (tj pouzivat treba Point k prenaseni H/W jenom proto ze to je take dva INTegery)
* v jave to jsou Point, Dimension, Rectangle ... (definovany jako mutable)

# Sluzebnik - Servant
* Pokud mame skupinu trid (nemusi mit nutne stejneho predka - kdyby meli je lepsi definovat metody v predkovi) a chceme na nich **definovat** nejakou **schopnost** tak aby **nevznikli duplicity kodu** muzeme zavest Sluzebnika
* Nejlepe si ho popiseme na prikladu vytvoreni : 
  1. **Analizujeme ktere hodnoty potrebuje k provedeni akci** ktere maji mit objekty spolecne a co se zmeni na objektech (getPosition(), setPosition(Position p))
  2. Nechame objekty **implementovat rozhrani**
  3. Definujeme metodu tridy **Sluzebnik** ktera jako parametr bere objekt ktery implementuje nami vytvoreny interface
* Zpusob pouziti (ve smyslu kdo na koho vidi)
  * **Uzivatel vidi sluzebnika** a objekty trid -> preda sluzebnikovi jako parametr objekt tridy u ktere chce vyvolat pozadovane chovani
  * ![tutorialspoint](https://upload.wikimedia.org/wikipedia/commons/3/38/DesignPatternServantFigure1.png "Uzivatel vidi sluzebnika")
  * **Sluzebnika vidi objekty trid** na kterym ma bych chovani vyvolane -> zavolaji sluzebnika a jako parametr daji "this"
  * ![Wikipedie](https://upload.wikimedia.org/wikipedia/commons/4/41/DesignPatternServantFigure2.png "Sluzebnika vidi objekty trid")

# Prikaz - Command
![Wikipedie](http://www.tutorialspoint.com/design_pattern/images/command_pattern_uml_diagram.jpg "Prikaz")
* Podobne jako vzor Sluzebnik
* Mam skupinu objektu a chci je doplnit o nejakou funkcnost ( definuju rozhrani jehoz "instance" predavam puvodnim objektum jako parametry funkci
* Lze vytvoret **pole metod** (ktere implentuji definove rozhrani - zobrazeni cisel na kostce)
* Potrebuje zpetne volani
* https://en.wikipedia.org/wiki/Command_pattern#Java

# Prazdny objekt - Null object
* Slouzi k osetreni toho aby nenastavala NullPointerException a jine podobne vyjimky (tim ze priradime do promenne "null"). 
 1. Tvori se tak ze vezmeme objekt a vsechny jeho metody presunume do interface (nebo abstraktni tridy) a nas soucasny objekt bude implementovat (resp. dedit) chovani + vytvorime NullObject tohoto rozhrani kde v impementaci metod osetrime null (nejcasteji budou mit prazdne telo a nebo vracet dohodnutou hodnotu) -> budem ale muset predavat predka (interface)
 2. Nebo budeme prazdny objekt dedit od pozadovane tridy bez vytvareni abstrace (popr. rozhrani) a jenom prekryjeme metody
*  Prazdny objekt by mel byl navrzen **jako singleton** (o tom pozdeji)
*  ![tutorialspoint](http://www.tutorialspoint.com/design_pattern/images/null_pattern_uml_diagram.jpg "Prazdny objekt")

# Jedinacek - Singleton
* V cele aplikaci se vyskutuje pouze jedna unikatni instance teto tridy (zajisteno pomoci **privatniho konstruktoru, staticke promenne a jednoduche tovarni funkci**)
* Sva druhy inicializace
  * **Casna inicializace**, tj. v deklaraci promenne (budeme preferovat private - pokud public tak musime vzit v uvahu ze virtualni stroj zavede tridy a inicializuje promenne ve spravnem poradi) + getSignleton() 
   ```Java
   private static final MySingleton mySingleton = new MySingleton();
   ```
  * **Odlozena inicializace** (lazy inicialization) - inicializuje az kdyz je potreba (vetsinou v getSignleton()) - lepsi pro narocnejsi inicializace - nemusi se take provest, kdyz neni potreba a usetri se vykon
* pozor u **multi-thread aplikaci a lazy inicialization** - treba dvoji check proti null + synchronizace (co nejmensiho bloku kvuli vykonu) + u promenne uvest **volatile** (prekladace se nebudou snazit o zjednoduseni a pri kazde potrebe instanci znova nactou a ne z cache (registrech))
* pozor take na **serializovatelnost** - sla by tak vytvorit kopie objektu (to je proti zasade Jedinacka) - musime osetrit metodu **readResolve()** a nebo neserializovat
* lze vyuzit dedicnosti a jako sinleton vracet nektereho z potomku (pokud nejdou zanorene musime ozelet privatnosti konstruktoru)

# Vyctovy typ (Enumerated type)
* Lze pouzit jako klasicke vyctove pole bez dalsich rozsireni
* Lze vytvorit privatni konstuktor a jako staticke verejne promnenne vyctoveho typu definuje primo instance tohoto vyctoveho typu - uchovana hodnota navic
   ```Java
   public enum Month {
    JANUARY("Je-li teplo v lednu, sahá bída ke dnu."),
    FEBRUARY("Únor bílý, pole sílí.");
    private String weatherLore;

    private Month(String weatherLore) {
        this.weatherLore = weatherLore;
    }
    ...
    }
   ```
* Pokud definujeme jako funkcni vyctovy typ ktery reaguje jinak na nejakou metodu - lze ulozit hodnotu navic + prvky polymorfismu
  ```Java
   public enum Month {
    JANUARY{
       public void sayName(){System.out.println("I am January")};
    },
    FEBRUARY{
       public void sayName(){System.out.println("I am February")};
    };

    private Month() {}
    abstract public void sayName();
    ...
    }
   ```
   pote k tomu pristupujeme pres rodice (predavame promnenou typu Month)

#Musi vaha - Flyweight - GOF
* Pokud se **pro urcitou skupinu objektu** (musi jich byt mnoho, aby se to vyplatilo) dari najit urcitou **skupinu "reprezentantu"** pak je tento pattern uzitecny (vykreslovani 20 kruhu o 5 barvach - neni nutne mit 20 objektu ale jen 5)
   * **interni** charakteristiky - spolecne pro urcitou skupinu - bude interni pro reprezentanta
   * **externi** charakteristiky - to co se bude menit (budem setovat potomi metod na instanci)
* Hodi se vyvorit nejakou tridu ktera bude hlidat kteri reprezentanti uz jsou vytvoreni (pripadne je vracet) a v pripade potreby vytvaret nove
* ![tutorialspoint](http://www.tutorialspoint.com/design_pattern/images/flyweight_pattern_uml_diagram.jpg "Musi vaha")
