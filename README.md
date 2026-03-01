# OVF Predictor

Herramienta de apoyo a la decisión clínica para fracturas osteoporóticas toracolumbares.

## Modelo

- **Algoritmo**: Regresión Logística (L2, C=1.0)
- **AUC**: 0.655 (10-fold stratified cross-validation)
- **Dataset**: 482 pacientes
- **Variables**: 18 predictores clínico-radiológicos

## Funcionamiento

Calcula la probabilidad de resultado clínico-radiológico favorable en dos escenarios:
1. Tratamiento conservador
2. Cementación vertebral

La diferencia estima el beneficio potencial de la intervención.

## Variables del modelo

| Variable | Coef. | Dirección |
|----------|-------|-----------|
| VA (Cementación) | +0.565 | → Bueno |
| Sin traumatismo | -0.378 | → Malo |
| Tipo fractura OF | -0.351 | → Malo |
| Lugar residencia | -0.265 | → Malo |
| Vida indep. previa | +0.264 | → Bueno |
| Fractura múltiple | -0.239 | → Malo |
| Tto. antiOP previo | -0.222 | → Malo |
| Osteoporosis Dx | +0.194 | → Bueno |
| IMC | -0.191 | → Malo |
| UH (Hounsfield) | +0.125 | → Bueno |
| Sexo (masculino) | +0.121 | → Bueno |
| Segmento torácico | +0.105 | → Bueno |
| Calcio sérico | +0.105 | → Bueno |
| ACO | -0.094 | → Malo |
| Trauma baja energía | -0.080 | → Malo |
| ASA Score | -0.074 | → Malo |
| Edad | -0.067 | → Malo |
| Fragilidad (mFI) | +0.043 | → Bueno |

## Despliegue en Vercel

1. Sube este repositorio a GitHub
2. Conecta en [vercel.com](https://vercel.com) → Import Project
3. Deploy automático

## Disclaimer

**Solo para uso en investigación.** Prototipo académico.

## Autores

Martínez-Núñez P, Gutiérrez-González R — Universidad Europea — TFM 2025/2026
