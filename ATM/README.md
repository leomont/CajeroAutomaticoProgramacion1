# Proyecto ATM — Descripción completa

Este documento describe en detalle el proyecto "ATM" (ejemplo didáctico de un cajero automático), su estructura de código, flujo de ejecución, modelo de datos, casos límite, instrucciones de compilación/ejecución y mejoras recomendadas.

---

## Resumen

ATM es una aplicación Java standalone que simula las operaciones básicas de un cajero automático: autenticación de usuario mediante número de cuenta y PIN, consulta de saldo, retiros y depósitos. El proyecto es una implementación educativa basada en el ejemplo clásico de Deitel & Associates. Está diseñado para ejecutarse en consola y usar componentes simulados de "hardware" (pantalla, teclado, dispensador de efectivo, ranura de depósito).

---

## Estructura de ficheros (principal)

Ubicación: `src/main/java`

Clases principales y responsabilidades:

- `ATMCaseStudy.java`
  - Clase con el método `public static void main(String[] args)` que inicia la aplicación (arranca una instancia de `ATM` y llama a `run`).

- `ATM.java`
  - Controlador principal. Orquesta la autenticación, muestra el menú principal, crea y ejecuta transacciones (`Transaction`), y coordina con `BankDatabase` y componentes hardware simulados.

- `BankDatabase.java`
  - Simula la base de datos de cuentas en memoria (`Account[]`). Provee métodos para autenticar, obtener saldos y modificar balances (`authenticateUser`, `getAvailableBalance`, `getTotalBalance`, `credit`, `debit`).

- `Account.java`
  - Modelo de dominio que almacena `accountNumber`, `pin`, `availableBalance` y `totalBalance`. Provee validación de PIN y operaciones `debit`/`credit`.

- `Transaction.java`
  - Clase base/abstracta para transacciones. Define la interfaz de una transacción y referencias a `Screen`, `BankDatabase`, etc.

- `BalanceInquiry.java`
  - Implementa `Transaction` para mostrar saldos.

- `Withdrawal.java`
  - Implementa la lógica de retiro: comprobar fondos, verificar disponibilidad del `CashDispenser` y debitar la cuenta.

- `Deposit.java`
  - Implementa la lógica de depósito: simula la inserción de sobre y acredita la cuenta si el `DepositSlot` confirma la recepción.

- `Screen.java`
  - Abstracción responsable de mostrar mensajes al usuario (salida por consola).

- `Keypad.java`
  - Abstracción responsable de leer entradas del usuario (teclado/console).

- `CashDispenser.java`
  - Simula el dispensador de billetes: mantiene un stock de billetes y puede determinar si puede dispensar una cantidad.

- `DepositSlot.java`
  - Simula la ranura de depósito y confirma la recepción de sobres.

---

## Flujo de ejecución (típico)

1. `ATMCaseStudy.main` crea una instancia de `ATM` y llama a `run()`.
2. `ATM.run()` muestra mensaje de bienvenida y llama a `authenticateUser()` hasta que el usuario se autentica correctamente.
3. Tras autenticación, se muestra el menú principal (ver saldo, retirar, depositar, salir).
4. Según la opción, `ATM` crea una instancia del tipo de `Transaction` correspondiente y llama a `execute()`.
5. La transacción realiza consultas y/o modificaciones en `BankDatabase` y puede interactuar con `CashDispenser` o `DepositSlot`.
6. El usuario puede realizar múltiples operaciones en una sesión; al salir la sesión termina y el ciclo puede reiniciarse.

---

## Modelo de datos (contrato mínimo)

- Account:
  - accountNumber: int
  - pin: int
  - availableBalance: double
  - totalBalance: double
  - métodos: `validatePIN(int)`, `getAvailableBalance()`, `getTotalBalance()`, `credit(double)`, `debit(double)`

- BankDatabase: métodos clave pública
  - `boolean authenticateUser(int accountNumber, int pin)`
  - `double getAvailableBalance(int accountNumber)`
  - `double getTotalBalance(int accountNumber)`
  - `void credit(int accountNumber, double amount)`
  - `void debit(int accountNumber, double amount)`

- Transaction: cada transacción tiene referencia al número de cuenta, `Screen` y `BankDatabase` y un método `execute()`.

---

## Cómo compilar y ejecutar

Opciones (elige la que corresponda en tu entorno):

1) Con Maven (si está instalado):

- Compilar y ejecutar tests:

```bash
mvn clean test
```

- Compilar y empaquetar:

```bash
mvn clean package
```

- Ejecutar la aplicación desde clases compilas (si el `main` es `ATMCaseStudy`):

```bash
mvn compile
mvn exec:java -Dexec.mainClass="ATMCaseStudy"
```

2) Alternativa sin Maven (javac + java):

- Compilar todas las clases y colocar clases en `target\classes` (Windows cmd.exe):

```bash
mkdir target\classes
javac -d target\classes src\main\java\*.java
```

- Ejecutar la clase principal (suponiendo `ATMCaseStudy` tiene `main`):

```bash
java -cp target\classes ATMCaseStudy
```

Nota: en entornos donde `mvn` no está disponible, la alternativa con `javac` funciona siempre que tengas JDK instalado.

---

## Casos límite, riesgos y observaciones

- Persistencia: `BankDatabase` usa un arreglo en memoria. Los cambios no persisten entre ejecuciones.
- Búsqueda: `Account[]` hace búsquedas lineales; usar `Map<Integer, Account>` mejora complejidad.
- NullPointerException: algunos métodos de `BankDatabase` llaman a `getAccount(...)` sin comprobar nulos — riesgo de NPE si se usan accountNumber inválidos.
- Concurrencia: no hay sincronización; acceso concurrente a las cuentas puede causar condiciones de carrera.
- Seguridad: no hay límite de intentos de PIN ni bloqueo de cuentas.
- Validaciones: falta validación de montos (negativos, precisión, límites) y sanitización de entrada.
- Simulación de hardware: `CashDispenser` y `DepositSlot` son simulaciones simples; el manejo de errores físicos es limitado.

---

## Mejoras recomendadas (priorizadas)

1. Persistencia ligera: serializar cuentas a JSON/CSV o usar BD embebida (H2/SQLite).
2. Reemplazar `Account[]` por `Map<Integer, Account>` para acceso O(1).
3. Manejar retornos nulos en `BankDatabase` y lanzar excepciones claras o usar `Optional<Account>`.
4. Añadir límite de intentos de PIN y bloqueo temporal de cuentas.
5. Añadir pruebas unitarias (JUnit) para `BankDatabase`, `Account`, `Withdrawal` y `Deposit`.
6. Añadir logging (SLF4J + Logback) para auditoría de transacciones.
7. Validación robusta de inputs (montos, rangos) y manejo de errores de hardware simulado.
8. Mejorar UX de consola o exponer un API REST para integración.

---

## Pruebas sugeridas

- Pruebas unitarias (JUnit):
  - Autenticación correcta e incorrecta.
  - Debitar con fondos suficientes y con fondos insuficientes.
  - Depositar y comprobar saldo total/available.
  - Simular fallo de `CashDispenser` (fondos insuficientes en hardware).

- Tests de integración: flujo completo (autenticación → retiro → balance) en modo headless usando entradas simuladas.

---

## Contribuir

- Clona el repositorio, crea una rama por función/bugfix y abre un pull request describiendo el cambio.
- Añade tests que cubran nueva funcionalidad o correcciones.

---

## Atribución y licencia

Este código se basa en el ejemplo educativo de Deitel & Associates y Pearson Education (comentarios de copyright y disclaimer incluidos en varios archivos fuente). Conserva la atribución existente en los archivos fuente.

---

## Cobertura de la solicitud

- Requisito: "genera un archivo md. con la descripcion completa del proyecto ATM" — Hecho: este archivo `ATM_PROJECT_DESCRIPTION.md` contiene la descripción completa solicitada, instrucciones de compilación, riesgos y mejoras.

---

Si quieres que añada este contenido como `README.md` en vez de `ATM_PROJECT_DESCRIPTION.md`, o que incluya ejemplos concretos de tests JUnit o un README más corto para usuarios, dime cuál prefieres y lo creo inmediatamente.

