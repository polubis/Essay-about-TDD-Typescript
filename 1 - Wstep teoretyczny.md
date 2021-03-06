# Essay-about-TDD-Typescript

## (Opcjonalnie) Krótko o teorii muzyki

https://github.com/polubis/Essay-about-TDD-Typescript/blob/main/TEORIA_MUZYKI.md

## Co implementujemy?

Aby przykład był trochę inny zajmiemy się implementacją prostej apki do wizualizacji dźwięków na gryfie gitary.

Jako użytkownik powinniśmy móc:

- Wyświetlić dźwięki gitary
- Mieć możliwość pokazania dźwięków tylko w jednym kolorze
- Pokazać wartość oktawy przy dźwięku bądź ją schować
- Zmienić typ notacji z # na b

Design:

![image](https://user-images.githubusercontent.com/22937810/151938836-3658e938-22db-4206-8561-8d0aeec13d6f.png)

## Przykład z życia

Jesteś pracownikiem pizzeri. Pizzeria ma swój proces tworzenia pizzy. Początkowo jesteś tylko uczniem. Patrzysz jak pizza jest przygotowywana przez zawodowców. Każdy z nich zajmuje się innym fragmentem procesu, który posiada następujące fazy.
  
 1. Wyrobienie ciasta oraz jego wyrost (Jaro).
 2. Przygotowanie składników (Don Vasyl).
 3. Stworzenie pizzy (Śledziu).
 
Każda z tych osób przeprowadza swojego rodzaju testy. Przykładowo **Śledziu** sprawdzi sprężystość ciasta podczas próby jego formowania, **Don Vasyl** sprawdzi czy składniki nie są za grubo pokrojone, a **Jaro** sprawdzi temperaturę kawałka.

Gdyby pominąć faze testowania może się zdarzyć, że klient dostanie złej jakości produkt. Cały proces kończy się pojawieniem się pizzy na talerzu klienta oraz jej spróbowaniem.

Teraz wyobraź sobie kod. Pracownicy to moduły, które komunikują się ze sobą, a każdy z nich posiada inną odpowiedzialność. Czy rozsądnym jest zakładanie, że kod będzie zawsze działał? Od tego mamy testy automatyczne, które będą pełniły role pracowników pizzeri.

## Czym jest test?

Test to nic innego jak weryfikacja czy nasz kod działa według zdefiniowanych wymagań. Można testować manualnie / automatycznie. Standardowy podział testów w oparciu o testowany obszar to:

- Jednostkowe
- Integracyjne
- e2e

Istnieją jeszcze inne podziały, o tym później.

## Narzędzia i technologie

- **jest** - framework do testowania.
- **react-testing-library** - libka do testowania komponentów w **React**.
- **react-testing-library/hooks** - rozszerzenie do testowania **hooks w React**.
- **jsdom** - symulacja prawdziwego drzewa **DOM**. **jest** z tego korzysta.

## Arrange & Act & Assert

Podejście w którym staramy się rozdzielić 3 etapy (jeżeli jest taka możliwość - czasami testy są jednolinijkowe). Przygotowanie fragmentu kodu do testowania, uruchomienie testowanego kodu oraz sprawdzenie wyników.

```ts
it("goes through whole update procedure", async () => {
  // Arrange start
  const VALID_USER: User = { username: "Tomasz1994", id: 0 };
  fetch.mockResponseOnce(JSON.stringify(VALID_USER));
  const userService = new UserService();
  // Arrange end

  // Act start
  const user = await userService.updateUser(VALID_USER);
  // Act end

  // Assert start
  expect(fetch).toHaveBeenCalledTimes(1);
  expect(fetch).toHaveBeenCalledWith("/user", JSON.stringify(VALID_USER));
  expect(user).toEqual(VALID_USER);
  // Assert end
});
```

## Testy jednostkowe

Pod uwagę bierzemy tylko jedno konkretne zachowanie kodu. Przekładając to na nasz przykład testujemy albo wilgotność ciasta albo jego rozmiar.

1. Funkcja `sum()` powinna dodawać do siebie podane parametry i zwracać wynik.
 
```ts
// sum.ts
// Prosta funkcja sumująca parametry
const sum = (...args: number[]) => args.reduce((acc, nmb) => acc + nmb, 0);

// Test jednostkowy sprawdzający poprawność działania funkcji sum()
it('returns the sum of parameters', () => {
  expect(sum(3,45,2)).toBe(50)
});
// W tym teście testujemy tylko i wyłącznie zwracany rezultat
```

2. Komponent Button powinien przypisać motyw oraz wyświetlić dowolny podany kontent wewnątrz.

![Przycisk](https://www.fibaro.com/en/wp-content/uploads/sites/3/2017/02/color-bt-red.png)

```ts
// Button.tsx
enum ButtonTheme {
  Primary = 1,
  Secondary = 2,
}

interface ButtonProps {
  children: ReactNode;
  theme: ButtonTheme;
}

const Button = ({ children, theme }: ButtonProps) => {
  return (
    <button role="button" className={css[`theme${theme}`]}>
      {children}
    </button>
  );
};
```

```ts
// Button.test.tsx
// Testowanie wyświetlania zawartości
it('displays content', () => {
  render(<Button theme={ButtonTheme.Primary}>Content</Button>);
  expect(screen.getByText(/content/)).toBeInTheDocument();
});

// Testowanie motywu
it('assigns theme', () => {
  render(<Button theme={ButtonTheme.Primary}>Content</Button>);

  const { background, color } = screen.getByText(/content/).style;

  expect(background).toBeTruthy();
  expect(color).toBeTruthy();
});
```

Zauważ, że napisaliśmy 2 oddzielne testy. Każdy z nich testuje inny fragment funkcjonalności.

## Testy integracyjne

Testujemy większy fragment funkcjonalności. Przekładając to na nasz przykład testujemy jak zachowa się ciasto po przejściu przez proces formowania i wyrastania.

```ts
// user.ts
interface User {
  id: number;
  username: string;
}

// user.model.ts
class UserModel implements User {
  constructor(public id: number, public username: string) {}

  isValid = (): boolean => {
    return this.id > 0 && this.username.length > 8;
  };
}

// user.service.ts
class UserService {
  updateUser = (user: User): Promise<User> | never => {
    const userModel = new UserModel(user.id, user.username);

    if (!userModel.isValid()) {
      throw new Error("Invalid model");
    }

    return fetch("/user", { method: "post", body: JSON.stringify(user) }).then(
      (res) => res.json()
    );
  };
}

// user.service.test.ts
it("throws an error for invalid model", () => {
  const userService = new UserService();
  expect(() => userService.updateUser({ username: "", id: -1 })).toThrow();
});

it("goes through whole update procedure", async () => {
  const VALID_USER: User = { username: "Tomasz1994", id: 0 }; // MOCK
  fetch.mockResponseOnce(JSON.stringify(VALID_USER));
  const userService = new UserService();

  const user = await userService.updateUser(VALID_USER);

  expect(fetch).toHaveBeenCalledTimes(1);
  expect(fetch).toHaveBeenCalledWith("/user", JSON.stringify(VALID_USER));
  expect(user).toEqual(VALID_USER);
});
```

Testujemy wykorzystanie modułu z walidacją w serwisie.

## Testy e2e

Testujemy całość. Przekładając to na pizzerie. Klient je pizze. Przetestował cały proces. Nie interesuje go kto robił pizze, ile wyrastało ciasto, ...itd. Jeżeli mu smakuje to wszystko jest ok. Test procesu może też później odbywać się w WC.

```ts
 it('allows user to create account', () => {
   var page = 'https://www.some-page.com/account-creation'
   
   cy.page(page)
   cy.get('input[type="email"]').type('mail.com').should('have.value','mail.com');
   cy.contains('Invalid mail!');
   cy.get('input[type="email"]').type('piotr@gmail.com').should('have.value','piotr@gmail.com');
   cy.get('Invalid mail!').should('not.exists');
   cy.get('button[type="submit"]').click();
   cy.url().should('include', 'creating');
 })
```

## False negatives w testach

Sytuacja, w której testy wskazują błąd, a w rzeczywistości go nie ma. Błąd jest w teście, a nie w implementacji kodu.

```ts
// Button.tsx
import css from './Button.scss';

interface ButtonProps {
  className?: string;
  children: ReactNode;
}

const Button = ({ className = '', children }: ButtonProps) => {
  return (
      <button className={`${css.btn} ${className}`}>{children}</button>
  )
}

// Button.test.tsx

// Spowoduje false negatives
it('adds class', () => {
    // Ten test się zrobi czerwony jeżeli dodamy 3 klasę bądź damy klasę pod warunkiem
    const { container } = render(<Button className='my-class'>btn</Button>);
    expect(screen.getByText(/btn/).classList.length).toBe(2);
})
```

## False positivies

Sytuacja, w której wygląda na to, że wszystko jest ok, a w rzeczywistości nie jest. Również błąd w implementacji testu. 

```ts
const Parent = () => {
  return (
    <>
      <label>Labelka</label>
      <Child />
    </>
  )
}

const Child = () => {
  return (
    <button>Labelka</button>
  )
}

// Mamy 2x labelka tekst więc nawet jak usuniemy "Labelka" z Parent to dalej test przejdzie, a nie powinien.
it('displays labelka tekst', () => {
  render(<Parent />);
  expect(screen.getByText(/Labelka/)).toBeInTheDocument()
})
```

## Mocks = Dummies & Fakes & Stubs

### Dummy 

Zaspokajamy API bo i tak chcemy testować coś innego.

```ts
const renderNoteButtonComponent = (
  props: Partial<NoteButtonComponentProps> = {}
) => render(<NoteButtonComponent name="C" position={0} {...props} />);  
```

W testach nie istotne są **position** oraz **name** więc korzystamy z **dummy** i przekazujemy tam jakiekolwiek wartości.

### Fakes

Chcemy testować działanie aplikacji z w połączeniu z bazą danych. Podmieniamy prawdziwą bazę bazą in-memory z takim samym API.

### Stubs

Nadpisują metody w taki sposób, żeby zwracały one **hard coded values** na potrzeby testu.

### Mocks

To co wyżej + również mogą nadpisywać i ingerować w kolejność działań. 
Często wszystkie te pojęcia grupuje się w jedno określenie - poprostu **Mock**.

## O tym jak skrupulatne testowanie chroni developerów

Wyobraź sobie duża aplikacje z brakiem jakichkolwiek testów, a teraz obok weż pod uwagę taką samą z testami jednostkowymi, które wykryją problem z konkretnym modułem, integracyjnymi które wykryją nie działający moduł oraz e2e, które otwierając okno przeglądarki wykonają za użytkownika test czy aplikacja w swoim naturalnym środowisku działa poprawnie.

## Test Driven Development

Jest to podejście, w którym najpierw powinniśmy:
1. Napisać wymagania w dowolnej formie.
2. Określić zakres funkcjonalności, która mamy aktualnie implementować (nie całość naraz).
3. Szkielet rozwiązania. 
4. Wybieramy fragment funkcjonalności - granulacja pracy.
  5. **Faza red** Stworzyć interfejsy, napisać testy, które nie przechodzą (są czerwone).
  6. **Faza green** Poprawić kod tak, aby testy, które napisaliśmy działały.
  7. **Faza refactor** - zrobić refactor kodu.
  8. Jeżeli po 7 kroku testy się popsuły to wracamy do kroku 6.
9. Przejść do kroku 4 lub jeżeli to już wszystko to kończymy.

![Przycisk](https://s3.amazonaws.com/mokacoding/2018-09-18-red-green-refactor.jpg)

Oficjalnie tylko kroki **5,6,7** należą do **TDD**. Jednak postanowiłem umieścić je wewnątrz całego procesu developmentu, aby pokazać, w którym miejscu powinniśmy korzystać z **TDD**. W **TDD** powinniśmy się trzymać korków **5,6,7** wszystkie pozostałe sprawdzają się u mnie. Dla każdego może to być troche inny proces.

## TDD w praktyce

// TODO
// Dodac memy
// Zrobic rewizje dokumentu
// Dac faq
/// zrobic prezke
// Dokonczyc apke

### 1. Napisać wymagania w dowolnej formie.

Na samym początku dokumentu.

### 2. Określić zakres funkcjonalności, która mamy aktualnie implementować

To czym w pierwszej kolejności warto się zająć to punkt **Wyświetlić dźwięki gitary.** Reszta funkcjonalności jest zbudowana na wizualizacji więc od tego zaczniemy.

### 3. Szkielet rozwiązania

Pisząc aplikacje warto rozważyć rozdzielenie domeny biznesowej od prezentacji. Przykładowo zamiast kodować logikę w komponentach **React**, można zaimplementować jej w warstwie abstrakcji, która jest po za **React**. Takie rozwiązanie sprawi możliwość łatwego otestowania najważniejszej części funkcjonalności czyli logiki po za frameworkiem.

Dodatkowo zyskujemy też możliwość łatwego przejścia na inną technologie, użycie logiki oraz zrozumiały podział na logikę i prezentacje. 

Zatem stworzymy katalog o nazwie **music-core**. Znajdzie się tam wszystko co dotyczy teorii muzyki, liczenia skal, dźwięków, ...itd. Będzie to w przyszłości idealny kandydat na  bibliotekę.

Drugi katalog nazwiemy **fretboard-visualization** i umieścimy go w katalogu **modules**. Będzie to katalog zawierający komunikację pomiędzy kompontentami za pomocą kontekstu, obsługę stanu oraz wszystkie komponenty prezentacyjne dotyczące tej funkcjonalności. 

W katalogu **modules** będziemy umieszać w przyszłości również inne większe, podzielne funkcjonalności. Można to traktować jako katalog grupujący małe aplikacje. Wewnątrz tych modułów zgrupujemy sobie pliki po ich przeznaczeniu. Komponenty do katalogu **components**, rzeczy związane z komunikacją pomiędzy komponentami do katalogu **providers**, komponenty połączone za pomocą **Context API** do katalogu **containers** oraz całą logikę dotyczącą konkretnej funkcjonalności do katalogu **models**. 

### 4. Wybieramy fragment funkcjonalności - granulacja pracy

Warto zacząć od rzeczy najmniejszych i stopniowo iść do góry. Zaczniemy od implementacji przycisku pozwalającego na przedstawienie dźwięków.

![image](https://user-images.githubusercontent.com/22937810/151938883-27a6eeec-0f39-4564-936e-c577f29dbd0c.png)
![image](https://user-images.githubusercontent.com/22937810/151938928-f72a560d-8b36-4b9e-93a4-be89cf130368.png)

### 5. **Faza red** Stworzyć interfejsy, napisać testy, które nie przechodzą (są czerwone)

Nasz **NoteButtonComponent** będzie miał następujący interfejs:

```ts
// NoteButtonComponent.tsx
const NOTE_POSITIONS = [0,1,2,3,4,5,6,7,8,9,10,11] as const;
const NOTE_OCTAVE = [0,1,2,3,4,5,6,7,8] as const;

type NotePosition = typeof NOTE_POSITIONS[number];
type NoteOctave - typeof NOTE_OCTAVE[number];

export interface NoteButtonComponentProps {
  className?: string;
  position: NotePosition;
  octave?: NoteOctave; 
  name: string;
  singleColor?: boolean;
  hidden?: boolean;
  onClick?: (e: React.MouseEvent<HTMLButtonElement, MouseEvent>) => void;
}
```

Następnie dodajemy nie zaimplementowany komponent i rzucamy wyjątek:

```ts
// NoteButtonComponent.tsx
const NoteButtonComponent = ({}: NoteButtonComponentProps) => {
  throw new Error('Not implemented yet');
}

export { NoteButtonComponent } 
```

W tym momencie mamy "zaspokojenie" zgodności typów. Napiszmy wszystkie scenariusze testowe według wymagań. Narazie nie implementujmy testów.

```ts
// NoteButtonComponent.test.tsx

describe('NoteButtonComponent', () => {
  it('displays only note name');
  it('displays note name with octave')
  it('calls parent function with dataset attributes');
})
```

Jak widać mamy 3 rzeczy warte przetestowania. Skupiamy się na tym, aby nazwy testów nie nawiązywały do szczegółów implementacyjnych (nie zawsze o tym później). Przykładowo:
```ts
it('assigns note color if singleColored property is falsy') // jeżeli usuniemy potrzebe użycia flagi singleColored to będziemy musieli zmienić również nazwę testów. Większe utrzymanie
```

Teraz implementacja testów:

```ts
// NoteButtonComponent.test.tsx
import {
  NoteButtonComponent,
  NoteButtonComponentProps,
} from "./NoteButtonComponent";
import { fireEvent, render, screen } from "@testing-library/react";

describe("NoteButtonComponent", () => {
  const renderNoteButtonComponent = (
    props: Partial<NoteButtonComponentProps> = {}
  ) => render(<NoteButtonComponent name="C" position={0} {...props} />);

  it("displays only note name", () => {
    renderNoteButtonComponent();
    expect(screen.getByText(/C/)).toBeInTheDocument();
    expect(screen.queryByText(/4/)).not.toBeInTheDocument();
  });

  it("displays note name with octave", () => {
    renderNoteButtonComponent({ octave: 4 });
    expect(screen.getByText(/C/)).toBeInTheDocument();
    expect(screen.getByText(/4/)).toBeInTheDocument();
  });

  it("calls parent function with dataset attributes", () => {
    const spy = jest.fn();
    const Stub = () => (
      <NoteButtonComponent
        name="C"
        position={0}
        octave={4}
        onClick={(e) =>
          spy({
            position: e.currentTarget.dataset.position,
            octave: e.currentTarget.dataset.octave,
          })
        }
      />
    );

    render(<Stub />);
    fireEvent.click(screen.getByText(/C/));

    expect(spy).toHaveBeenCalledTimes(1);
    expect(spy).toHaveBeenCalledWith({ position: "0", octave: "4" });
  });

  afterEach(() => {
    jest.clearAllMocks();
  });
});
```

### 6. **Faza green** Poprawić kod tak, aby testy, które napisaliśmy działały.

```ts
import { NoteOctave, NotePosition } from "music-core";
import { Button } from "antd";

import css from "./NoteButtonComponent.module.less";

export interface NoteButtonComponentProps {
  className?: string;
  position: NotePosition;
  octave?: NoteOctave;
  name: string;
  singleColor?: boolean;
  hidden?: boolean;
  onClick?: (e: React.MouseEvent<HTMLButtonElement, MouseEvent>) => void;
}

const COLORS = [
  "#f08989",
  "#cc9d72",
  "#a4cc72",
  "#72cc76",
  "#72ccbc",
  "#72b1cc",
  "#729bcc",
  "#9a72cc",
  "#728bcc",
  "#7f72cc",
  "#c572cc",
  "#cc72a8",
] as const;

const NoteButtonComponent = ({
  className = "",
  position,
  name,
  octave,
  singleColor,
  hidden,
  onClick,
}: NoteButtonComponentProps) => {
  return (
    <Button
      role="button"
      className={`${css.btn} ${className} ${hidden ? css.hidden : ""}`}
      style={{
        background: singleColor ? undefined : COLORS[position],
      }}
      type={singleColor ? "primary" : undefined}
      data-position={position}
      data-octave={octave}
      onClick={onClick}
    >
      <span className={css.name}>{name}</span>
      {octave !== undefined && <span className={css.octave}>{octave}</span>}
    </Button>
  );
};

export { NoteButtonComponent };
```

Dodaliśmy implementacje, która sprawia, że wcześniej napisane testy zaczynają być zielone.

### 7. **Faza refactor** - zrobić refactor kodu.

W tym kroku można modyfikować kod, wydzielać dodatkowe funkcje, komponenty dla czytelności, lepszego performance, ...itd.
Oczywiście robimy to mając informacje zwortną z działających w tle testów. Poniższej przykład:

![Example of TDD](https://user-images.githubusercontent.com/22937810/152313514-f98c597f-32bc-4db3-ab56-e7d8eb2adfc9.gif)

### 8. Jeżeli po 7 kroku testy się popsuły to wracamy do kroku 6.

Testy działają więc idziemy dalej.

### 9. Przejść do kroku 4 lub jeżeli to już wszystko to kończymy.

U nas to nie wszystko i powinniśmy wrócić do kroku 4. Potrzebujemy zaimplementować jeszcze wiele funkcjonalności. Ich kod można zobaczyć na tym branchu:

https://github.com/polubis/music-app/tree/Release-1.5/apps/jam-jam

Proces tworzenia jednej większej znajdziesz tutaj:

https://github.com/polubis/Essay-about-TDD-Typescript/blob/main/2%20-%20TDD%20na%20wiekszym%20przykladzie.md

## Piramida testów

Jest to podejście, w którym najwięcej mamy testów jednostkowych, później integracyjnych, a na najmniej e2e. 

![Piramida testów](https://projectquality.it/wp-content/uploads/2020/02/Piramida-Testo%CC%81w-Projectquality.it_-1024x640.png)

To podejście jest czasochłonne. Piszemy wiele testów. Jest również podejście, w którym piszemy najwięcej testów jednostkowych bo to one mogą wykryć dziure w systemie o wiele łatwiej niż testy jednostkowe - testujemy większy obszar.
Takie testy częściej będą czerwone oraz wolniejsze jednak sprawdzą większa część systemu.

## Testowanie publicznego API

Powinniśmy starać się zawsze testować tylko publiczne API. Patrząc na przykład niżej metoda prywatna jest nie istotna z perspektywy modułu korzystającego z **UserModel**. Zazwyczaj jeżeli ktoś testuje metody czy pola prywatne jest to problem z designem kodu.

```ts
// user.ts
interface User {
  id: number;
}

class UserModel {
  private _setId(id: number): void {
    this.id = id;
  }
  
  public update(user: User): void {
    this._setId(user.id);
  }
}

// user.test.ts
it('updates user', () => {
  expect(new UserModel().update({id: 0} ).id).toBe(0);
})
```

Są jednak przypadki, że czasami trzeba to zrobić. Przykładowo podczas integracji z jakąś biblioteką.

## Co warto testować?

Rzeczy krytyczne dla systemu w pierwszej kolejności. Działanie autoryzacji, kluczowe aspekty funkcjonalności, logikę biznesową, zachowanie się komponentów. Pamiętamy o tym, że
testy piszemy dla stabilności rozwiązania oraz jest to narzędzie dające nam prostą odpowiedź na pytanie: Czy coś działa? Tak/Nie. 

## Czy zawsze warto testować?

Oczywiście, że nie. Robimy gdy mamy na to czas oraz gdy mamy określone, niezmieniające się wymagania (TDD) oraz kod nadający się do testowania. Kod, który się nie nadaje to funkcja, która ma 200 linijek i pełno warunków brzegowych, często też zagnieżdźonych.

## Testowanie szczegółów implementacyjnych

Czasami nie da się inaczej. Wyobraź sobię gotową bibliotekę do wysyłania maili. Najwygodniej jest przetestować czy konkretna metoda została z niej wywołana. Nie ma możliwości testowania mechanizmu ponieważ jest on przed nami ukryty.

Lub sytuacja, w której chcesz przetestować przypisanie klasy. Te testy też mają ogromną wartość poprostu mają tendencje do psucia się w przypadku zmiany implementacji. Przykładowo zmienisz implementację wyglądu z css na inline style i taki test już nie przechodzi. Daje **false negatives**.

## Code coverage

Przydatna rzecz jeżeli chcemy widzieć jakie fragmenty kodu brał udział w testach.

![image](https://user-images.githubusercontent.com/22937810/152534321-71f55783-1545-435c-8bc1-c056665af5b8.png)

