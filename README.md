# Sistema de Nómina

Proyecto académico de Ingeniería de Software · **Universidad de Cartagena – Centro Tutorial Mompox**
**CIPA: Compiladores del Futuro**

## 1. Objetivo
Registrar empleados de cuatro tipos y calcular su **salario bruto, bonos, beneficios, deducciones y salario neto**, demostrando Programación Orientada a Objetos (encapsulamiento, herencia, polimorfismo, abstracción), SOLID, código limpio, refactorización y pruebas unitarias.

## 2. Tecnologías
- Python 3.10+
- Tkinter (interfaz gráfica, incluida con Python)
- pytest (pruebas unitarias)

## 3. Instalación
```bash
git clone <url-del-repositorio>
cd sistema_nomina
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```
> En Linux, si falta Tkinter: `sudo apt install python3-tk`.

## 4. Ejecución
```bash
python -m src.main        # abre la interfaz gráfica
python -m pytest          # ejecuta las pruebas
```
En la interfaz: elija el tipo de empleado → llene los datos → **Calcular nómina**.
Escriba los números con punto para miles y coma para decimales (`3.000.000`, `40,5`). La comisión se escribe como porcentaje (`5` = 5 %). Si un dato es inválido, se muestra un mensaje en rojo.

## 5. Tipos de empleado
| Tipo | Resumen |
|---|---|
| Asalariado | Salario fijo mensual. Permanente (alimentación). Bono 10 % con más de 5 años. |
| Por horas | Tarifa por hora. Horas > 40 se pagan x1.5. Sin bonos. Fondo de ahorro opcional con más de 1 año. |
| Por comisión | Salario base + comisión sobre ventas. Permanente (alimentación). Bono 3 % de las ventas si superan $20.000.000. |
| Temporal | Salario fijo mensual, contrato por tiempo definido. Sin bonos ni beneficios. |

## 6. Reglas principales de negocio
- **Salario bruto** = salario base (incluye comisión u horas extras) + bonos.
- **Deducción** (Seguro Social y Pensión) = salario bruto × 4 %.
- **Salario neto** = salario bruto − deducciones. Nunca puede ser negativo.
- **Beneficio de alimentación**: $1.000.000 mensuales, cubierto por la empresa (asalariado y comisión).
- **Fondo de ahorro** (por horas): 2 % del salario, si tiene más de 1 año y lo acepta.
- **Validaciones**: no se aceptan salarios, tarifas, horas ni ventas negativas; la comisión debe estar entre 0 % y 100 %; la duración del contrato debe ser un entero positivo.
- **ARL**: la actividad no indica porcentaje, por lo que **no se calcula**. El porcentaje debe ser definido por el docente.

**Supuestos de interpretación** (para aclarar con el docente): los bonos suman al salario bruto; los beneficios (alimentación y fondo de ahorro) los cubre la empresa, por eso se muestran aparte y no cambian el salario neto, que se calcula exactamente como lo define la actividad (bruto − deducciones).

### Ejemplos
| Caso | Bruto | Deducción 4 % | Neto |
|---|---|---|---|
| Asalariado, $3.000.000, 6 años (bono $300.000) | $3.300.000 | $132.000 | $3.168.000 |
| Por horas, $20.000/h, 45 h (40 normales + 5 extras) | $950.000 | $38.000 | $912.000 |
| Comisión, base $2.000.000, ventas $25.000.000, 5 % (comisión $1.250.000 + bono $750.000) | $4.000.000 | $160.000 | $3.840.000 |
| Temporal, $2.500.000 | $2.500.000 | $100.000 | $2.400.000 |

Uso desde código:
```python
from src.models import EmpleadoAsalariado
from src.services import NominaService

empleado = EmpleadoAsalariado("1", "Ana", anios_antiguedad=6, salario_mensual=3_000_000)
resultado = NominaService().calcular_nomina(empleado)
print(resultado.salario_neto)   # 3168000.0
```

## 7. Principios SOLID aplicados
- **SRP**: los modelos guardan las reglas de cada empleado; `NominaService` calcula el resultado final; `validaciones.py` valida; `ui/` solo muestra datos; las pruebas están en `tests/`.
- **OCP**: para agregar un tipo nuevo se crea su clase y se registra en `tipos_empleado.py`. `NominaService` y la interfaz (que genera sus campos desde el registro) no se modifican. Hay una prueba que lo demuestra (`test_ocp_nuevo_tipo...`).
- **LSP**: cualquier subclase de `Empleado` se usa en `NominaService` sin romper nada (`test_polimorfismo...`).
- **DIP**: `NominaService` depende de la abstracción `Empleado`, no de clases concretas.
- **ISP**: no se crearon interfaces adicionales; `Empleado` es pequeña y toda subclase usa todos sus métodos (bonos y beneficios tienen valor por defecto vacío). No se forzó una abstracción artificial.

## 8. Estructura del proyecto
```
sistema_nomina/
├── src/
│   ├── models/        # Empleado (abstracta) y sus 4 tipos + validaciones
│   ├── services/      # NominaService, registro de tipos y conversión de datos
│   ├── ui/            # Interfaz Tkinter (sin lógica de negocio)
│   └── main.py
├── tests/             # Pruebas unitarias
├── README.md
├── requirements.txt
├── pytest.ini
└── .gitignore
```

## 9. Pruebas unitarias
`python -m pytest` ejecuta 36 pruebas: bono y beneficio del asalariado; horas normales/extras, horas negativas y fondo de ahorro; comisión, bono del 3 % y ventas negativas; temporal sin bonos ni beneficios; deducción del 4 %, salario neto y no negatividad; polimorfismo, OCP y conversión de datos del formulario.

## 10. Refactorización realizada
1. **Beneficio de alimentación duplicado** en Asalariado y Comisión → extraído a `EmpleadoPermanente`.
2. **Validaciones repetidas** en cada constructor → centralizadas en `validaciones.py`.
3. **Cálculo del bruto repetido** → un solo método plantilla `calcular_salario_bruto` en `Empleado`.
4. **Un `if/elif` por tipo en la interfaz** → formulario generado desde el registro `TIPOS_EMPLEADO`.
5. **Números "mágicos"** (40, 1.5, 0.04, 20.000.000…) → constantes con nombre.

## 11. Metodología de desarrollo
Scrum simplificado en iteraciones cortas: análisis de requisitos → diseño (diagrama de clases) → desarrollo → pruebas → refactorización → documentación → entrega. Cada etapa se refleja en uno o más commits.

**Secuencia de commits sugerida**
1. `chore: crear estructura inicial del proyecto`
2. `feat: crear clase abstracta Empleado y los cuatro tipos`
3. `feat: implementar reglas de nómina (bonos, beneficios, deducción 4 %)`
4. `feat: agregar validaciones de datos`
5. `refactor: aplicar SOLID y extraer EmpleadoPermanente`
6. `feat: crear interfaz gráfica`
7. `test: agregar pruebas unitarias`
8. `docs: README y entrega final`

## 12. Integrantes del CIPA
**CIPA: Compiladores del Futuro** · Universidad de Cartagena · Centro Tutorial Mompox

- *(Agregar nombre del integrante 1)*
- *(Agregar nombre del integrante 2)*
- *(Agregar nombre del integrante 3)*
