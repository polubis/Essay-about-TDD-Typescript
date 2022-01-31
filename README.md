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
3. Projektujemy strukturę plików, interfejsy, moduły, api. 
4. **Faza red** Napisać testy, które nie przechodzą (są czerwone).
5. **Faza green** Poprawić kod tak, aby testy, które napisaliśmy działały.
6. **Faza refactor** Zrobić refactor kodu.
7. Jeżeli po 5 kroku testy się popsuły to wracamy do kroku 4.
8. Przejść do następnego fragmentu całej funkcjonalności.

![Przycisk](https://s3.amazonaws.com/mokacoding/2018-09-18-red-green-refactor.jpg)

Oficjalnie tylko kroki **4,5,6** należą do **TDD**. Jednak postanowiłem umieścić je wewnątrz całego procesu developmentu, aby pokazać, w którym miejscu powinniśmy korzystać z TDD. Również powinniśmy się trzymać korków **4,5,6** wszystkie pozostałe sprawdzają się u mnie. Dla każdego może to być troche inny proces.

## TDD w praktyce

Aby przykład był trochę inny zajmiemy się implementacja prostej apki do wizualizacji dźwięków na gryfie gitary. Wykonujemy powyższe kroki.

**1. Napisać wymagania w dowolnej formie**.

Jako użytkownik powinniśmy móc:

- Wyświetlić dźwięki gitary.
- Mieć możliwość pokazania dźwięków tylko w jednym kolorze.
- Pokazać wartość oktawy przy dźwięku bądź ją schować.
- Zmienić typ notacji z # na b.

TODO: Tu wstawić design

**2. Określić zakres funkcjonalności, która mamy aktualnie implementować**.

To czym w pierwszej kolejności warto się zająć to punkt **Wyświetlić dźwięki gitary.** Reszta funkcjonalności jest zbudowana na wizualizacji więc warto od tego zacząć.

**3. Projektujemy strukturę plików, interfejsy, moduły, api.**


