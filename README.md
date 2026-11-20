# BTCUSDT AI Trading Agent

**Agente de trading algorítmico con Deep Learning y un LLM local para el mercado BTCUSDT de MEXC Spot.**

Proyecto de inteligencia artificial financiera que integra un modelo predictivo LSTM, un modelo de lenguaje ejecutado localmente mediante Ollama y un sistema de gestión de riesgo para proponer y simular decisiones de trading.

El objetivo es construir un agente capaz de analizar información de mercado, generar pronósticos, proponer acciones, validar dichas acciones mediante controles deterministas de riesgo y ejecutar operaciones en un entorno de Paper Trading.

> **Estado del proyecto:** desarrollo inicial. La arquitectura y los componentes están definidos; las funcionalidades se implementarán y validarán progresivamente.

---

## Arquitectura

La arquitectura general del sistema es:

```text
                 USER
                  |
                  v
        React + TypeScript
          (Frontend / UI)
                  |
                  v
          FastAPI Backend
          (API autenticada)
                  |
          +-------+-------+
          |               |
          v               v
     LSTM Model        Ollama Local
   (Predicción)          (LLM)
          |               |
          +-------+-------+
                  |
                  v
          Risk Management
         (Reglas deterministas)
                  |
                  v
           Paper Trading
          (Simulación)
                  |
                  v
        SQLite / Persistence
                  |
                  v
          Dashboard / Logs
```

## Tecnologías

### Frontend — User Interface

- React
- TypeScript
- Vite
- [Tecnologías de visualización por definir]

La interfaz permitirá consultar el estado del agente, visualizar información de mercado, revisar pronósticos y decisiones, y controlar la ejecución mediante las opciones de iniciar, pausar y detener.

### Backend

- Python
- FastAPI
- PyTorch
- Pandas
- NumPy
- Scikit-learn
- SQLite

El backend será responsable de:

- Consultar datos públicos del mercado MEXC.
- Validar y procesar velas de mercado.
- Calcular las variables requeridas por el modelo.
- Cargar el modelo predictivo y sus transformaciones.
- Coordinar la interacción con Ollama.
- Validar las acciones propuestas.
- Aplicar controles de riesgo.
- Ejecutar operaciones simuladas.
- Registrar decisiones, operaciones y resultados.

### Local LLM

- Ollama
- Modelo de lenguaje descargado y ejecutado localmente.

El LLM tendrá la función de analizar información estructurada y proponer acciones de trading. Su acceso estará protegido mediante una API autenticada en el backend.

El modelo de lenguaje no tendrá acceso a claves privadas ni autoridad para modificar o evadir los controles deterministas de riesgo.

**Requisito:** las inferencias del LLM deberán realizarse localmente, sin depender de servicios de inferencia en la nube.

### Market Data

- MEXC API
- BTCUSDT Spot
- Velas diarias (UTC)

Se utilizarán datos públicos de mercado. El sistema deberá verificar que las velas estén cerradas y que la información corresponda al periodo esperado.

### Trading Simulation

- Paper Trading
- Simulador propio
- Sin envío de órdenes reales

Las operaciones serán simuladas con un saldo virtual. Se considerarán el efectivo disponible, la posición, los costos y las restricciones de negociación.

---

## Modelo predictivo

El proyecto parte del trabajo desarrollado en la práctica de Deep Learning para Finanzas.

| Característica | Configuración |
|---|---|
| Activo | BTCUSDT |
| Mercado | MEXC Spot |
| Frecuencia | Diaria |
| Modelo | LSTM |
| Arquitectura | 1 capa LSTM + Linear |
| Hidden size | 32 |
| Ventana de entrada | 20 rendimientos |
| Semillas | 101, 202, 303 |
| Predicción | Rendimiento del siguiente periodo |

El modelo utiliza un ensemble de tres redes LSTM entrenadas con diferentes semillas.

Se conservarán los checkpoints y los escaladores aprobados, respetando el contrato de variables y las transformaciones utilizadas durante el entrenamiento.

La integración deberá verificar la compatibilidad entre los datos de entrada, el escalador y la arquitectura del modelo.

---

## Funcionamiento esperado del agente

El ciclo de decisión estará compuesto por las siguientes etapas:

1. Obtener las velas cerradas de BTCUSDT desde MEXC.
2. Validar fechas, integridad y antigüedad de los datos.
3. Calcular las variables de entrada.
4. Obtener el pronóstico del modelo LSTM.
5. Enviar información estructurada a Ollama.
6. Recibir una propuesta de acción en formato JSON.
7. Validar la respuesta y las acciones permitidas.
8. Aplicar los controles deterministas de riesgo.
9. Ejecutar la operación en el simulador Paper.
10. Registrar la decisión y actualizar el estado de la cartera.
11. Mostrar los resultados en la interfaz.

El agente deberá conservar su estado y evitar procesar dos veces la misma vela.

---

## Gestión de riesgo

El sistema incorporará controles independientes del LLM:

- Sin apalancamiento.
- Sin posiciones cortas.
- Límite máximo de exposición.
- Límite de pérdida diaria.
- Validación del saldo disponible.
- Validación del tamaño y precisión de las operaciones.
- Control de antigüedad de los datos.
- Bloqueo de nuevas operaciones cuando se alcance un límite.
- Control de operaciones duplicadas.

Los parámetros de riesgo no podrán ser modificados por el LLM.

El botón de detener bloqueará nuevas decisiones y operaciones. La liquidación de una posición existente se manejará mediante una regla explícita.

---

## Persistencia

Se utilizará SQLite para conservar información relacionada con:

- Velas procesadas.
- Pronósticos.
- Decisiones del agente.
- Órdenes simuladas.
- Costos de operación.
- Efectivo y posiciones.
- Evolución del patrimonio.
- Errores y eventos del sistema.

La persistencia deberá permitir recuperar el estado del agente después de reiniciar el backend sin duplicar operaciones.

---

## Seguridad

- Las claves y credenciales se almacenarán exclusivamente en el backend.
- Los secretos no se incluirán en el repositorio.
- Se proporcionará un archivo `.env.example` sin valores reales.
- Las rutas de control del agente requerirán autenticación.
- Se rechazará el acceso mediante credenciales inválidas.
- No se habilitarán operaciones reales en MEXC.
- El LLM no podrá modificar los controles de riesgo.

---

## Manejo de errores

Se probarán, como mínimo, los siguientes escenarios:

1. API de mercado sin respuesta o con límite de solicitudes.
2. Datos atrasados.
3. Servicio Ollama detenido.
4. Respuesta JSON inválida.
5. API key incorrecta.
6. Saldo insuficiente.
7. Evento duplicado.
8. Reinicio del backend.
9. Activación del botón de detener.

El sistema deberá registrar los errores, mostrar mensajes útiles y evitar operaciones indebidas.

---

## Evaluación

Se evaluará el comportamiento del sistema mediante:

- Al menos 20 ciclos válidos de decisión.
- Comparación de la estrategia predictiva sin LLM.
- Comparación del agente con LLM.
- Comparación contra Buy & Hold.
- Rendimiento neto.
- Maximum Drawdown.
- Win Rate.
- Número de operaciones.
- Latencia.
- Acciones rechazadas por los controles de riesgo.

El objetivo de la evaluación es determinar si la integración del LLM aporta valor al proceso de decisión. No se presupone que necesariamente mejore los resultados.

---

## Estructura prevista del repositorio

```text
btc-trading-agent/
│
├── frontend/
│   ├── src/
│   ├── public/
│   └── package.json
│
├── backend/
│   ├── app/
│   │   ├── api/
│   │   ├── core/
│   │   ├── models/
│   │   ├── services/
│   │   ├── risk/
│   │   ├── trading/
│   │   └── database/
│   └── requirements.txt
│
├── artifacts/
│   └── README.md
│
├── tests/
│
├── docs/
│
├── .env.example
├── .gitignore
└── README.md
```

La estructura se irá ajustando conforme se implementen y prueben los componentes.

---

## Instalación y ejecución

Las instrucciones definitivas de instalación y ejecución se documentarán conforme se integren el frontend, backend y servicio local de Ollama.

Los requisitos previstos son:

- Python
- Node.js y npm
- Ollama
- SQLite
- Dependencias de Python y JavaScript del proyecto

---

## Seguridad y alcance

Este proyecto es de carácter académico y experimental.

Todas las operaciones se realizarán en un entorno de simulación. No se enviarán órdenes reales a MEXC ni se administrarán fondos reales.

Los resultados históricos y las predicciones no constituyen garantías de rendimiento futuro.

---

## Autoría

Proyecto desarrollado como parte de la práctica de Deep Learning para Finanzas.

**Activo:** BTCUSDT  
**Exchange:** MEXC Spot  
**Enfoque:** Deep Learning, LLM local, gestión de riesgo y Paper Trading.
