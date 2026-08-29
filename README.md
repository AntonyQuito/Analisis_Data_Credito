# Analisis_Data_Credito

## Resumen
*(Escribe aquí una breve descripción general de tu proyecto, qué herramientas utilizaste y qué problema estás resolviendo con el análisis de datos).*

## Objetivo de Proyecto 
*(Define cuál es la meta principal que buscas alcanzar con este análisis financiero o de créditos).*

## Preguntas

**1. ¿Cuántos son los clientes que tienen más edad?**
```sql
Select
         edad,
         COUNT(*) as Cantidad_Clientes
From [Proyecto_Personal].[dbo].[clientes_credito]
Group by edad
Order By edad



## Conclusiones 
* Conclusión 1 basada en los hallazgos de tus consultas SQL y datos.
* Conclusión 2.

## Recomendaciones 
* Recomendación 1 orientada al negocio o a la toma de decisiones.
* Recomendación 2.
