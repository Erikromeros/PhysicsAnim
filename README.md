# Normas de desarrollo

Este documento establece las normas de estilo y mejores prácticas para el código y blueprints en este proyecto. Es crucial seguir estas directrices para mantener el código limpio, consistente y fácil de mantener.

# Estructura de Carpetas ejemplo "Avión"

Para asegurar la coherencia y facilidad de mantenimiento del proyecto, es crucial organizar adecuadamente todos los archivos y recursos relacionados con el actor principal, el avión. A continuación, se detalla la estructura de carpetas recomendada para todo lo relacionado con este componente:

## Estructura General

- **Avion/**
  - **Animaciones/**: Todas las animaciones específicas del avión, incluyendo despegues, aterrizajes, maniobras, etc.
  - **Meshes/**: Contiene todos los modelos 3D, incluyendo variantes y componentes.
	- **Static Mesh o Malla Estatica**: Inician con la nomenclatura **SM_**
	- **Skeletal Mesh o Malla esquelética**: Inician con la nomenclatura **SK_**
  - **Blueprint/**: Aquí se almacenan los scripts específicos del avión, como la lógica de vuelo, control de armamento, etc.
	- **Blueprint**: Inician con la nomenclatura **BP_**
  - **Materials/**: Contiene todos los materiales, instancias, funciones materiales y coleciones.
	- **Master Material**: Inician con la nomenclatura **MM_**
	- **Material Base**: Inician con la nomenclatura **M_**
	- **Material Instanciado**: Inician con la nomenclatura **MI_**
	- **funciones materiales**: Inician con la nomenclatura **MF_**
  - **Sonidos/**: Archivos de audio utilizados por el avión, como el ruido del motor, efectos de disparo, comunicaciones, etc.
	- **Sound Wave**: Inician con la nomenclatura **SW_**
	- **Sound Cue**: Inician con la nomenclatura **SC_**
  - **UI/**: Elementos de la interfaz de usuario relacionados específicamente con el avión, como HUDs, indicadores de daño, combustible, etc.
	- **Widget Blueprint**: Inician con la nomenclatura **WB_**
  - **Textures/**: contiene todas las texturas, Inician con la nomenclatura  **T_** y finaliza con la nomenclatura al que pertence, ejemplo **T_Window_D**
	- **Diffuse, Albedo o Base Color**: seran nombrados con la letra **_D**
	- **Normal**: seran nombrados con la letra **_N**
	- **Metallic o metalidad**: seran nombrados con la letra **_M**
	- **Roughness o Rugosidad**: seran nombrados con la letra **_R**
	- **Ambient Oclusion**: seran nombrados con la letra **_AO**
	- **Height o Altura**: seran nombrados con la letra **_H**
	- **Opacity o Opacidad**: seran nombrados con la letra **_OP**
	- **Emissive o Emisivo**: seran nombrados con la letra **_E**
	- **Masks o Máscaras**: seran nombrados con la letra **_M**
  - **Documentacion/**: Documentos técnicos y guías relacionadas con el avión, incluyendo manuales de operación, especificaciones técnicas, En WORD, etc.	

## Nomenclatura

- **Clases:** Utilizar UpperCamelCase para los nombres de clases. Ejemplo: `MiClase`.
- **Variables:** Utilizar lowerCamelCase para los nombres de variables. Ejemplo: `miVariable`.
- **Funciones:** Las funciones deben ser claras y descriptivas, utilizando UpperCamelCase. Ejemplo: `CalcularDistancia`.

## Comentarios

- **Comentar Código:** Todo código o blueprint significativo debe ser adecuadamente comentado. Los comentarios deben explicar el "por qué" detrás del código, no el "qué" que ya se explica con el código en sí.
- **Secciones Inmutables:** Para aquellas funciones o secciones de blueprint que no deben ser modificadas directamente (por ejemplo, partes de un plugin que deben permanecer intactas), incluir un comentario claramente visible que indique:
     `plaintext`
  // NO MODIFICAR: Esta sección es parte del plugin XYZ. Para extensiones o modificaciones, copiar e instanciar como un nuevo elemento.
    
## Trabajo con Plugins
Al trabajar con plugins, no modificar directamente el código o blueprints del plugin. Si necesitas extender o cambiar la funcionalidad, copia el componente necesario y realízalo como una instancia separada o extiéndelo mediante herencia, según sea apropiado.
Parámetros de Instancia: Al instanciar componentes o clases del plugin, asegúrate de configurar correctamente todos los parámetros necesarios y documenta cualquier suposición o decisión importante.

## Contribuciones
Todas las contribuciones al proyecto deben seguir estas guías de estilo.
Antes de realizar un commit, revisa tus cambios para asegurarte de que se adhieran a estas normas.
Si introduces una nueva herramienta o patrón que requiera una actualización de estas guías, por favor propón los cambios necesarios a través de un pull request.

## Comandos de depuracion 
- **stat fps**
- **stat unit**
- **stat unitgraph**
- **stat engine**
- **stat gpu**
- **sstat cpuload**
- **stat streaming**
