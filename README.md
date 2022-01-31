# Essay-about-TDD-Typescript

## Wstęp

  Wyobraź sobie, że jesteś pracownikiem pizzeri. Mają skomplikowany proces tworzenia ciasta, jego wyrastania, pozyskiwania składników oraz ich obróbki.
  Początkowo jesteś tylko uczniem. Patrzysz jak pizza jest przygotowywana przez zawodowców. Każdy z nich zajmuje się innym fragmentem procesu oraz dba o jego prawidłowość, który posiada następujące fazy.
  
 1. Wyrobienie ciasta oraz jego wyrost (Jaro).
 2. Przygotowanie składników (Don Vasyl).
 3. Stworzenie pizzy (Śledziu).
 
  Każda z tych osób przeprowadza swojego rodzaju testy. Przykładowo **Śledziu** sprawdzi sprężystość ciasta podczas próby jego formowania, **Don Vasyl** sprawdzi czy składniki nie są za grubo pokrojone, a **Jaro** sprawdzi temperaturę kawałka.

  Gdyby pominąć faze testowania może się zdarzyć, że klient dostanie złej jakości produkt. 

## Przykłady testów

  Test to nic innego jak weryfikacja czy nasz kod działa według zdefiniowanych wymagań. Przykładowo zakładamy, że funkcja `sum()` powinna dodawać do siebie podane parametry i zwracać wynik. Zazwyczaj piszemy testy do gotowego już kodu.
 
1. Testy jednostkowe.
 
```ts
// sum.ts
// Prosta funkcja sumująca parametry
const sum = (...args: number[]) => args.reduce((acc, nmb) => acc + nmb, 0);

// Test jednostkowy sprawdzający poprawność działania funkcji sum()
it('returns the sum of parameters', () => {
  expect(sum(3,45,2)).toBe(50)
});
```

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

![Przycisk](https://www.fibaro.com/en/wp-content/uploads/sites/3/2017/02/color-bt-red.png)

```ts
// Button.test.tsx
it('displays content', () => {
  render(<Button theme={ButtonTheme.Primary}>Content</Button>);
  expect(screen.getByText(/content/)).toBeInTheDocument();
});

it('assigns theme', () => {
  render(<Button theme={ButtonTheme.Primary}>Content</Button>);

  const { background, color } = screen.getByText(/content/).style;

  expect(background).toBeTruthy();
  expect(color).toBeTruthy();
});
```

2. Testy integracyjne.

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
