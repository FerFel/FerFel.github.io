[gimmick: math]()

# Integralrechnung

## Stammfunktion

Die Stammfunktion zur Potenzfunktion 
$$
f(x)=x^n
$$
ist allgemein
$$
F(x)=\frac{1}{n+1} x^{n+1}
$$

Note:
Der Exponent wird um 1 erhöht und in den Nenner des Bruchs geschrieben

## Unbestimmtes Integral

Als unbestimmtes Integral bezeichnet man die Gesamtheit aller Stammfunktionen.
$$
\int f(x) dx = F(x) + C
$$
- f(x) ist der Integrand
- x die Integrationsvariable#
- C die Integrationskonstante

## Bestimmtes Integral
Wenn **Integrationsgrenzen** angegeben sind, handelt es sich nicht mehr um ein unbestimmtes Integral.
Im Gegensatz zum unbestimmten Integral lässt sich ein bestimmtes Integral mit dem **Hauptsatz der Integralrechnung** lösen.
Als Ergebniss erhält man einen kokreten Zahlenwert.
$$
\int_{a}^{b} f(x) dx=[F(x)]_a^b=(F(b)-F(a))
$$

# Integrationsregeln

### Potenzregel
$$
\int x^n dx = \frac{1}{n+1} x^{n+1} + C
$$

### Faktorregel
Ein konstanter Faktor kann vor das Integralzeichen gezogen werden.
$$
\int c \cdot f(x) dx = c \cdot  \int f(x) dx
$$

### Summenregel
Das unbestimmte Integral einer Summe ist gleich der Summe der unbestimmten Integrale.
$$
\int (f(x)) + g(x)) dx = \int f(x)dx + \int g(x) dx
$$

### Differenzregel
Das unbestimmte Integral einer Differenz ist gleich der Differenz der unbestimmten Integrale
$$
\int (f(x) - g(x)) dx = \int f(x) dx - \int g(x) dx
$$

## Partielle Integration
Was beim Ableiten die Produktregel ist, bezeichnet man beim Integrieren als partielle Integration.
$$
\int f'(x) g(x) dx = f(x)g(x) - \int f(x) g'(x) dx
$$
- es wird ein Faktor integriert
- und der andere Faktor abgeleitet

1. Die Ableitung welchen Faktors vereinfacht das Integral?
2. Stammfunktion des 1. Faktors berechnen
3. Ableitung des 2. Faktors berechnen
4. Ergebniss in Formel einsetzen

- Potenzen werden durch Ableiten einfacher
- Umkehfunktionen (ln, arcsin) werden durch Ableiten einfacher
- Funktionen wie e, sin werdern durch Integrieren nicht komplizierter
- manchmal hilft zweimaliges Integrieren und Umsortieren

## Integration durch Substitution
Was beim Ableiten die Kettenregel ist, bezeichnet man beim Integrieren als Substitutionsregel.
$$
\int f(x) dx = \int f(\varphi (u)) \cdot \varphi'(u) du
$$
