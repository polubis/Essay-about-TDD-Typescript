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

![alt text](https://raw.githubusercontent.com/username/projectname/branch/path/to/img.png)

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
