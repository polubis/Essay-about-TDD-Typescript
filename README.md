# Essay-about-TDD-Typescript

## Wstęp

  Wyobraź sobie, że jesteś pracownikiem pizzeri. Mają skomplikowany proces tworzenia ciasta, jego wyrastania, pozyskiwania składników oraz ich obróbki.
  Początkowo jesteś tylko uczniem. Patrzysz jak pizza jest przygotowywana przez zawodowców. Każdy z nich zajmuje się innym fragmentem procesu oraz dba o jego prawidłowość, który posiada następujące fazy.
  
 1. Wyrobienie ciasta oraz jego wyrost (Jaro).
 2. Przygotowanie składników (Don Vasyl).
 3. Stworzenie pizzy (Śledziu).
 
  Każda z tych osób przeprowadza swojego rodzaju testy. Przykładowo **Śledziu** sprawdzi sprężystość ciasta podczas próby jego formowania, **Don Vasyl** sprawdzi czy składniki nie są za grubo pokrojone, a **Jaro** sprawdzi temperaturę kawałka.

  Gdyby pominąć faze testowania może się zdarzyć, że klient dostanie złej jakości produkt. 

## Przykład testu

  Test to nic innego jak weryfikacja czy nasz kod działa według zdefiniowanych wymagań. Przykładowo zakładamy, że funkcja `sum()` powinna dodawać do siebie podane parametry i zwracać wynik.
 
1. Testy jednostkowe.

1a. Testowanie prostej funkcji.
 
```ts
// Implementacja
const sum = (...args: number[]) => args.reduce((acc, nmb) => acc + nmb, 0);
// Test
it('returns the sum of parameters', () => {
  expect(sum(3,45,2)).toBe(50)
});
```
