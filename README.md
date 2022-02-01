# Essay-about-TDD-Typescript

## Wstęp

  Wyobraź sobie, że jesteś pracownikiem pizzeri. Mają skomplikowany proces tworzenia ciasta, jego wyrastania, pozyskiwania składników oraz ich obróbki.
  Początkowo jesteś tylko uczniem. Patrzysz jak pizza jest przygotowywana przez zawodowców. Każdy z nich zajmuje się innym fragmentem procesu oraz dba o jego prawidłowość, który posiada następujące fazy.
  
 1. Wyrobienie ciasta oraz jego wyrost (Jaro).
 2. Przygotowanie składników (Don Vasyl).
 3. Stworzenie pizzy (Śledziu).
 
  Każda z tych osób przeprowadza swojego rodzaju testy. Przykładowo **Śledziu** sprawdzi sprężystość ciasta podczas próby jego formowania, **Don Vasyl** sprawdzi czy składniki nie są za grubo pokrojone, a **Jaro** sprawdzi temperaturę kawałka.

  Gdyby pominąć faze testowania może się zdarzyć, że klient dostanie złej jakości produkt. 

  Cały proces kończy się pojawieniem się pizzy na talerzu klienta oraz jej spróbowaniem.

## Przykłady testów

  Test to nic innego jak weryfikacja czy nasz kod działa według zdefiniowanych wymagań. Czasami wymagania znamy wcześniej, a czasami dochodzimy do nich podczas pracy nad kodem.
  
### Testy jednostkowe

Pod uwagę bierzemy tylko jedno konkretne zachowanie kodu. Porównując to do przykładu z początku testujemy tylko wilgotność ciasta lub tylko jego rozmiar.

1. Funkcja `sum()` powinna dodawać do siebie podane parametry i zwracać wynik. Zazwyczaj piszemy testy do gotowego już kodu.
 
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

### Testy integracyjne

Testujemy większy fragment funkcjonalności. Porównując do przykładu z życia testujemy po kolei _Wyrobienie ciasta oraz jego wyrost, Przygotowanie składników, Stworzenie pizzy_.

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
  const VALID_USER: User = { username: "Tomasz1994", id: 0 };
  fetch.mockResponseOnce(JSON.stringify(VALID_USER));
  const userService = new UserService();

  const user = await userService.updateUser(VALID_USER);

  expect(fetch).toHaveBeenCalledTimes(1);
  expect(fetch).toHaveBeenCalledWith("/user", JSON.stringify(VALID_USER));
  expect(user).toEqual(VALID_USER);
});
```

Testujemy wykorzystanie modułu z walidacją w serwisie.

### Testy e2e

Testujemy całość. Porównując do przykładu z życia testujemy _Stworzenie pizzy oraz jej smak_, ale nie w przyjaznym środowisku tylko już podczas ruchu - pora obiadowa.

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

## Test Driven Development

Jest to podejście, w którym najpierw powinniśmy:
1. Napisać wymagania w dowolnej formie.
2. Określić zakres funkcjonalności, która mamy aktualnie implementować (nie całość naraz).
3. Szkielet rozwiązania. 
4. Wybieramy fragment funkcjonalności - granulacja pracy.
  5. **Faza red** Stworzyć interfejsy, napisać testy, które nie przechodzą (są czerwone).
  6. **Faza green** Poprawić kod tak, aby testy, które napisaliśmy działały.
  7. **Faza refactor** Zrobić refactor kodu.
  8. Jeżeli po 7 kroku testy się popsuły to wracamy do kroku 6.
9. Przejść do kroku 4 lub jeżeli to już wszystko to kończymy.

![Przycisk](https://s3.amazonaws.com/mokacoding/2018-09-18-red-green-refactor.jpg)

Oficjalnie tylko kroki **5,6,7** należą do **TDD**. Jednak postanowiłem umieścić je wewnątrz całego procesu developmentu, aby pokazać, w którym miejscu powinniśmy korzystać z TDD. Również powinniśmy się trzymać korków **5,6,7** wszystkie pozostałe sprawdzają się u mnie. Dla każdego może to być troche inny proces.

## TDD w praktyce

Aby przykład był trochę inny zajmiemy się implementacja prostej apki do wizualizacji dźwięków na gryfie gitary. Wykonujemy powyższe kroki.

### 1. Napisać wymagania w dowolnej formie

Jako użytkownik powinniśmy móc:

- Wyświetlić dźwięki gitary.
- Mieć możliwość pokazania dźwięków tylko w jednym kolorze.
- Pokazać wartość oktawy przy dźwięku bądź ją schować.
- Zmienić typ notacji z # na b.

Design:

![image](https://user-images.githubusercontent.com/22937810/151938836-3658e938-22db-4206-8561-8d0aeec13d6f.png)

### 2. Określić zakres funkcjonalności, która mamy aktualnie implementować

To czym w pierwszej kolejności warto się zająć to punkt **Wyświetlić dźwięki gitary.** Reszta funkcjonalności jest zbudowana na wizualizacji więc warto od tego zaczniemy.

### 3. Szkielet rozwiązania

Pisząc aplikacje warto rozważyć rozdzielenie domeny biznesowej od prezentacji. Przykładowo zamiast kodować logikę w komponentach **React**, można zaimplementować jej w warstwie abstrakcji, która jest po za **React**. Takie rozwiązanie sprawi możliwość łatwego otestowania najważniejszej części funkcjonalności czyli logiki po za frameworkiem.

Dodatkowo zyskujemy też możliwość łatwego przejścia na inną technologie, użycie logiki oraz zrozumiały podział na logikę i prezentacje. 

Zatem stworzymy katalog o nazwie **music-core**. Znajdzie się tam wszystko co dotyczy teorii muzyki, liczenia skal, dźwięków, ...itd. Będzie to w przyszłości idealny kandydat na standalone bibliotekę.

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

interface NoteButtonComponentProps {
  className?: string;
  position: NotePosition;
  octave?: NoteOctave; 
  name: string;
  singleColor?: boolean;
  hidden?: boolean;
  onClick: (e: React.MouseEvent<HTMLButtonElement, MouseEvent>) => void;
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
  it('assigns note color');
  it('makes button hidden');
  it('displays note name');
  it('displays note octave')
  it('calls parent function with note position attribute');
  it('assigns color different than note color');
})
```

Jak widać w chwili obecnej mamy 5 rzeczy wartych przetestowania. Skupiamy się na tym, aby nazwy testów nie nawiązywały do szczegółów implementacyjnych. Przykładowo:
```ts
it('assigns note color if singleColored property is falsy') // jeżeli usuniemy potrzebe użycia flagi singleColored to będziemy musieli zmienić również nazwę testów. Większe utrzymanie
```



