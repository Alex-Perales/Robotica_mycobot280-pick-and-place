# 🤖 MyCobot 280 — Cinemática, Control y Visión Computacional

> **Examen Grupal — ROB 2026**
> Universidad ESAN · Facultad de Ingeniería
> Curso: *Cinemática, Control y Visión de Robots*
> Docente: **Calderón Niquin, Marks**

Proyecto integrador donde un brazo robótico serial de 6 grados de libertad (Yahboom MyCobot 280) detecta un objeto con una cámara, calcula su trayectoria mediante cinemática inversa y ejecuta un ciclo autónomo de **pick-and-place** sin intervención humana.

---

## 📌 Resumen del Proyecto

El objetivo es **integrar cinemática, control de trayectorias y visión computacional** en un solo pipeline autónomo que corra sobre el MyCobot 280, controlado desde una Jetson Nano con Ubuntu 20.04 y comunicado por USB a 1 000 000 baudios.

El sistema implementa una **máquina de estados** que recorre:

```
IDLE → DETECTANDO → CALC_IK → AGARRANDO → DEPOSITAR → IDLE
```

Cada estado tiene su propia lógica de error y vuelve automáticamente a una pose segura si algo falla.

---

## 🔗 Enlaces del Proyecto

| Recurso | Link |
|---|---|
| 📄 Informe técnico (Google Docs) | [Abrir documento](https://docs.google.com/document/d/1Gks6e2Yqms03d-Lj3XEG-5QRQKDgJ7Lu/edit?usp=sharing&ouid=104171794363611649136&rtpof=true&sd=true) |
| 🎨 Presentación (Canva) | [Ver presentación](https://canva.link/mcv00te7nqpdbef) |

---

## 👥 Equipo

| Integrante | Rol |
|---|---|
| Leon Condori Jesús Daniel | Líder de Integración |
| Rojas Estrada Julia Jhamilett | Especialista en Cinemática FK |
| Jordy Maximo Diaz Huanca | Especialista en Cinemática IK |
| Alex Perales Maldonado | Ingeniero de Control y Colisiones |
| Tovar Landa Henry Aaron | Ingeniero de Visión Computacional |

> Todos los integrantes ejecutaron llamadas directas al API del robot real con el hardware físico, tal como exige la consigna del examen.

---

## 🛠️ Stack Tecnológico

- **Hardware:** Yahboom MyCobot 280 (6-DOF serial) + Jetson Nano + cámara USB
- **Sistema:** Ubuntu 20.04 + ROS 2 Foxy
- **Comunicación:** Serial `/dev/ttyUSB0` @ 1 000 000 baud
- **Lenguaje:** Python 3 (Jupyter Notebook)
- **Librerías:** `pymycobot`, `OpenCV`, `NumPy`, `Matplotlib`, `ipywidgets`

---

## 📂 Estructura del Repositorio

```
.
├── main.py                          # Pipeline E2E con máquina de estados
├── cinematica.py                    # FK, IK, ik_solve y CollisionChecker
├── control.py                       # goto_pose, pick, place, run_cycle
├── vision.py                        # detect_object (HSV + píxel→mm)
├── notebooks/
│   ├── Rol1_Main.ipynb              # Integración (Leon)
│   ├── Rol2_Cinematica.ipynb        # FK + IK + Colisiones (Julia y Jordy)
│   ├── Rol3_Control.ipynb           # Control de trayectorias (Alex)
│   └── Rol4_Vision.ipynb            # Visión computacional (Aaron)
└── docs/
    ├── Informe_Tecnico.pdf
    ├── Presentacion.pdf
    └── Video_Demo.mp4
```

---

## ⚙️ Instalación y Ejecución

### 1. Requisitos previos
```bash
sudo apt update
sudo apt install python3-pip
pip install pymycobot opencv-python numpy matplotlib ipywidgets jupyter
```

### 2. Conectar el robot y la cámara
- Conectar el MyCobot 280 por USB → confirmar que aparece como `/dev/ttyUSB0`
- Conectar la cámara → confirmar que aparece como `/dev/video0`
- Dar permisos al puerto serie:
```bash
sudo chmod 666 /dev/ttyUSB0
```

### 3. Probar conexión
```python
from pymycobot.mycobot import MyCobot
import time

mc = MyCobot('/dev/ttyUSB0', 1000000)
mc.power_on()
time.sleep(1)
assert mc.is_controller_connected(), "Robot no conectado"
print("Ángulos:", mc.get_angles())
print("Coordenadas:", mc.get_coords())
```

### 4. Ejecutar el pipeline completo
```bash
jupyter notebook notebooks/Rol1_Main.ipynb
```
Ejecutar las celdas en orden. La celda final corre **5 ciclos autónomos** sin intervención humana.

---

## 🧩 Problemas Resueltos

| # | Problema | Lidera | Estado |
|---|---|---|---|
| P1 | Representación DH del MyCobot 280 | Julia / Jordy | ✅ |
| P2 | Cinemática Directa (FK) | Julia | ✅ |
| P3 | Cinemática Inversa (IK) | Jordy | ✅ |
| P4 | Evasión de colisiones | Alex | ✅ |
| P5 | Control de trayectorias (5 ciclos) | Alex | ✅ |
| P6 | Detección de objetos por visión | Aaron | ✅ |
| P7 | Pipeline End-to-End autónomo | Leon | ✅ |

---

## 📐 Parámetros DH del MyCobot 280

| Articulación | a (mm) | d (mm) | α | θ | Rango (°) |
|---|---|---|---|---|---|
| J1 — Base | 0 | 131.56 | +90° | θ₁ | −168 a 168 |
| J2 — Hombro | 110.4 | 0 | 0° | θ₂ | −135 a 90 |
| J3 — Codo | 96.0 | 0 | 0° | θ₃ | −150 a 150 |
| J4 — Muñeca 1 | 0 | 66.39 | −90° | θ₄ | −145 a 145 |
| J5 — Muñeca 2 | 0 | 73.18 | +90° | θ₅ | −165 a 165 |
| J6 — Gripper | 0 | 48.6 | 0° | θ₆ | −180 a 180 |

Alcance máximo extendido: **≈ 280 mm**.

---

## 🎯 Resultados Principales

| Métrica | Resultado |
|---|---|
| Error promedio FK vs robot real | < 10 mm |
| Error promedio IK (J1, J2, J3) | < 5° |
| Error promedio detección visión | < 5 mm |
| Tasa de éxito 5 ciclos E2E | **100% (5/5)** |
| Tiempo promedio por ciclo | ≈ 27 segundos |
| Altura mínima segura (Z_MIN) | 125 mm |

---

## 🔌 API de Referencia (`pymycobot`)

```python
mc.send_angles([j1,j2,j3,j4,j5,j6], speed)    # Mover por ángulos
mc.send_coords([x,y,z,rx,ry,rz], speed, mode)  # Mover por cartesianas (IK del firmware)
mc.get_angles()                                # Leer ángulos actuales
mc.get_coords()                                # Leer posición del extremo
mc.set_gripper_value(100, 80)                  # Abrir gripper
mc.set_gripper_value(0, 80)                    # Cerrar gripper
mc.is_controller_connected()                   # Verificar conexión
mc.power_on()                                  # Energizar motores
mc.release_all_servos()                        # Modo libre
```

---

## 🛡️ Estrategia de Evasión de Colisiones (P4)

Se eligió un enfoque **simple, justificado y efectivo** para el escenario del laboratorio:

1. **Límites articulares conservadores** — antes de cada `send_angles`, se verifica que los 6 ángulos estén dentro del rango DH.
2. **Verificación de altura mínima** — después de cada movimiento, se lee `get_coords` y si Z < 125 mm el robot vuelve automáticamente a `pose_inicial`.

Ambas verificaciones están encapsuladas en `goto_pose()`, la única función oficial de movimiento del sistema, garantizando que **ninguna parte del código pueda saltarse la seguridad por descuido**.

---

## ⚠️ Normas de Seguridad

- Nunca dejar el robot operando sin supervisión humana.
- Siempre iniciar en `pose_inicial`.
- Velocidad limitada a **20–30%** durante pruebas iniciales.
- Botón de parada de emergencia accesible desde Jupyter (`ipywidgets`).
- Verificación visual en modo lento antes de ejecutar a velocidad normal.

---

## 📋 Cronograma Cumplido

| Día | Actividad |
|---|---|
| Lunes | Setup + configuración del API + arquitectura |
| Martes | DH, poses base, calibración HSV, configuración cámara |
| Miércoles | FK + IK + agarre primario + detección de centroide |
| Jueves | Verificación + ciclo de agarre + integración visión-robot |
| Viernes | Pruebas E2E reales con el robot |
| Fin de semana | Ajuste final + grabación video + defensa oral |

---

## 🔮 Mejoras Futuras

- IK numérica iterativa propia (Jacobiano + pseudoinversa) para no depender del API en orientaciones complejas.
- Calibración de cámara con patrón de tablero de ajedrez para corregir distorsión de lente.
- Detector basado en deep learning ligero (YOLOv5n / MobileNet) para múltiples objetos sin recalibrar HSV.

---

## 📚 Referencias

- Craig, J. J. (2018). *Introduction to Robotics: Mechanics and Control* (4ª ed.). Pearson.
- Spong, M. W., Hutchinson, S., & Vidyasagar, M. (2020). *Robot Modeling and Control* (2ª ed.). Wiley.
- Siciliano, B., Sciavicco, L., Villani, L., & Oriolo, G. (2010). *Robotics: Modelling, Planning and Control*. Springer.
- Elephant Robotics. *MyCobot 280 User Manual & pymycobot API*. <https://docs.elephantrobotics.com/>
- OpenCV Documentation. <https://docs.opencv.org/>
- Denavit, J., & Hartenberg, R. S. (1955). *A kinematic notation for lower-pair mechanisms based on matrices*. ASME J. Applied Mechanics, 22, 215–221.

---

## 📝 Licencia

Proyecto académico desarrollado en el marco del curso **ROB 2026** — Universidad ESAN.
Uso libre con fines educativos. Atribución requerida al equipo autor.

---

<p align="center">
  <strong>Universidad ESAN · ROB 2026 · Lima, Perú</strong><br>
  <em>"Todos los integrantes deben poder explicar cualquier parte del sistema."</em>
</p>
