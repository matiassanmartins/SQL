# CASO CAMPUS
<p align="center">
  <img src="https://raw.githubusercontent.com/matiassanmartins/SQL/refs/heads/main/CONSULTA/CASO_CAMPUS/IMAGENES/LOGO.png" width="350" title="hover text">
</p>

**SHOOTOUT** es una empresa dedicada al arriendo de canchas de futbol, ubicada en el sector sur oriente de la Región Metropolitana.
El gran logro de esta empresa es el haber sido uno de los primeros centros deportivos en consolidar un sistema de programación de partidos de futbol en sus canchas para todo publico y con diferentes categorías de equipos.
La empresa además de su efectividad en la programación del uso de sus canchas, se caracteriza por entregar información de los partidos que se realizan en sus locaciones a los equipos que las utilizan. Dicha información, muchas veces es relevante para estos equipos, ya que a menudo son empresas las que logran formarlos dentro de la organización, para formar campeonatos internos que se juegan en las canchas de **SHOOTOUT**.
Por los motivos descritos, se le ha contactado para reestructurar y automatizar la extracción de información, con el objetivo de brindar a sus clientes un servicio mas completo y detallado de los partidos que se juegan en el recinto.

1.- La base del negocio es el tener la información de la programación y de los equipos que han realizado reservado las canchas, para disponer de esta información, se le solicita extraer programación de los partidos, en el formato y orden descrito a continuación. Debe indicar el estado de la programación como NO JUGADO (‘NJ’), en caso contrario la información del estado debe aparecer como JUGADO, además los datos deben estar ordenados por fecha ascendente y  hora ascendente.

select equipo_1 || ' vs ' || equipo_2 as "PARTIDO",
substring(equipo_1,1,3) || ' vs ' || substring(equipo_2,1,3) AS "ABREVIATURA",
to_char(fecha,'DD/MM/YY') as "FECHA PARTIDO",
to_char(hora,'HH24:MI') as "INICIO",
case estado 
when 'NJ' then 'NO JUGADO'
when 'J' then 'JUGADO' end as "ESTADO"
from programacion
order by "FECHA PARTIDO" asc,"INICIO" asc;

<p align="center">
  <img src="https://raw.githubusercontent.com/matiassanmartins/SQL/refs/heads/main/CONSULTA/CASO_CAMPUS/IMAGENES/RESPUESTA1.PNG" title="hover text">
</p>

2.- Así como se muestra la programación, también es relevante detallar la información de los partidos, sin embargo, en este segmento se consideran solamente los partidos jugados entre el 25 y 29 de agosto, con sus respectivos resultados, además se debe detallar el equipo que juega de local, el que juega de visita, la cancha y la fecha del encuentro. Como dato adicional debe mencionar al ganador del encuentro (LOCAL, VISITA) o mencionar EMPATE en caso que el marcador esté igualado, los datos deben ser ordenados por fecha del partido descendente y  equipo local ascendente.

select 
to_char(fecha,'DD/MM/YY') as "FECHA",
cancha as "CANCHA",
equipo_local as "LOCAL",
equipo_visita as "VISITA",
goles_local||'-'||goles_visita as "MARCADOR FINAL",
case
when goles_local>goles_visita then 'LOCAL'
when goles_local<goles_visita then 'VISITA'
else
'EMPATE'
end as "GANADOR"
from partido
where fecha between '25/08/18 00:00' and '29/08/18 23:59'
order by fecha desc, equipo_local asc;

<p align="center">
  <img src="https://raw.githubusercontent.com/matiassanmartins/SQL/refs/heads/main/CONSULTA/CASO_CAMPUS/IMAGENES/RESPUESTA2.PNG" title="hover text">
</p>

3.- Entre las garantías que otorga SHOOTOUT, está la de mantener a la audiencia de los partidos 100% informada; para este efecto, se registran todos los eventos que ocurren en los partidos. Estos eventos, pueden ser vistos por la audiencia en tiempo real (real time)  y para que esto suceda, usted debe proveer al departamento de comunicaciones y RRSS, la información de los eventos ocurridos en los partidos, con el siguiente orden y formato: evento (minuto ‘, tipo evento, , equipo del jugador,  jugador con camiseta entre paréntesis ej: (11), partido (abreviado con las 3 primeras letras de cada equipo + V/S, ej: BRA v/s CHI ) y la fecha del partido. Solo debe listar eventos hasta  15 días de ocurridos, ordenados por fecha y minuto del evento descendente.

select
minuto||''' GOL PARA '||equipo_jugador||': '||substring(nombre_jugador,1,1)
||'.'||apellido_jugador||'('||camiseta||')' as "EVENTO",
substring(equipo_local,1,3) || ' v/s ' || substring(equipo_visita,1,3) AS "PARTIDO",
to_char(fecha,'DD/MM/YY')
from evento
--SIMULAREMOS QUE NOS ENCONTRAMOS EN UNA FECHA CERCANA PARA CUMPLIR LA CONDICION DE LOS 15 DIAS Y OBTENER DATOS
--PERO LA SENTENCIA REAL ES where fecha between current_date-15 and current_date;
where fecha between to_date('05/09/18 00:00','DD/MM/YY HH24:00')-15 and '05/09/18 23:59'
order by fecha asc, minuto asc;

<p align="center">
  <img src="https://raw.githubusercontent.com/matiassanmartins/SQL/refs/heads/main/CONSULTA/CASO_CAMPUS/IMAGENES/RESPUESTA3.PNG" title="hover text">
</p>

4.- En el ámbito administrativo, se requieren de ciertos indicadores para tomar decisiones y guiar el negocio a buenos puertos. Una de estas decisiones contempla el sistema de remuneraciones de los empleados de SHOOTOUT, los que comisionan un porcentaje de cada partido que les toca observar, esto debido a que son ellos los que registran la información de los eventos y estadísticas propias de los partidos. Para este efecto, se desea saber username del usuario que observa y toma registro de los partidos, la cantidad de partidos observados, minutos totales observados, y la comisión a percibir que corresponde al 10,2% de la suma de los valores finales cobrados de cada partido observado. Tenga en cuenta que la tarifa se aplica por cada 60 minutos de partido, o el proporcional en la ocupación de la cancha. Se debe extraer la información desde el primer al último día de agosto del año actual y se debe ordenar por comisión final ascendente.

select 
usu.username as "USUARIO",
count(par.id_user) as "PARTIDOS OBSERVADOS",
sum(pro.minutos_jugados) as "MINUTOS OBSERVADOS",
to_char(sum(pro.minutos_jugados::decimal*(tar.valor::decimal/60))*0.102,'$99,999') as "COMISION FINAL"
from partido par
inner join usuario usu on usu.id_user=par.id_user
inner join programacion pro on pro.id_programacion=par.id_programacion
inner join tarifa tar on tar.id_tarifa=par.id_tarifa
--AL IGUAL QUE EN LA RESPUESTA ANTERIOR SIMULAREMOS LA FECHA
--RESPUESTA REAL -> extract(month from pro.fecha)=8 and extract(year from pro.fecha)=extract(year from current_date)
where pro.minutos_jugados is not null and extract(month from pro.fecha)=8 and extract(year from pro.fecha)=2018
group by usu.id_user;

<p align="center">
  <img src="https://raw.githubusercontent.com/matiassanmartins/SQL/refs/heads/main/CONSULTA/CASO_CAMPUS/IMAGENES/RESPUESTA4.PNG" title="hover text">
</p>

5.- Para incentivar la participación de la comunidad y de los equipos que se han registrado en SHOOTOUT, la empresa ha decidido realizar un descuento a los equipos afiliados. El descuento consiste en que SHOOTOUT abonará el 15% del valor pagado en la última visita, al próximo valor final del arriendo siguiente, al equipo que convoque como local del encuentro. Para aplicar el descuento, se le solicita a usted extraer un informe que contenga los partidos con fecha y hora, equipos locales (L) y visita(V), valor pagado y abono que se realizará al equipo que pagará el arriendo. La oferta aplica para equipos locales que hayan rentado una cancha entre los días 25 al 29 de agosto del año 2018, y que hayan completado el partido, en caso que el partido no se haya completado, los valores se deben mostrar con un “$0”. Los partidos deben estar ordenados por fecha de forma descendente.

select
to_char(par.inicio,'DD/MM/YYYY HH24:MI') as "FECHA",
to_char(par.inicio,'HH24:MI') as "HORA",
par.equipo_local as "LOCAL",
par.equipo_visita as "VISITA",
to_char(pro.valor_pagado,'$99,999') as "VALOR_PAGADO",
case
when par.fin is null then '$ 0'
else to_char(pro.valor_pagado*0.15,'$9,999')
end as "ABONO PROMOCION"
from partido par
join programacion pro on pro.id_programacion= par.id_programacion
where par.inicio between '25/08/18 00:00' and '29/08/18 23:59'
order by par.inicio asc;

<p align="center">
  <img src="https://raw.githubusercontent.com/matiassanmartins/SQL/refs/heads/main/CONSULTA/CASO_CAMPUS/IMAGENES/RESPUESTA5.PNG" title="hover text">
</p>

6.- Por lo general, los equipos acostumbran a pagar un abono para reservar la cancha y al final del encuentro cancelan el saldo o remanente del valor total de la tarifa aplicada. Se ha detectado que en algunos casos, no se han informado los minutos jugados a las programaciones, por ende no se pueden obtener los saldos pendientes. Para mantener esto en monitoreo, se le solicita un informe que muestre la información de cada equipo que tiene un saldo pendiente. El informe debe contener nombre de los equipos, minutos jugados, tarifa aplicada, total a pagar, monto abonado y saldo a pagar. En caso de que los minutos jugados no se hayan encontrado, se deben mostrar como “NO INFORMADOS” y el saldo a pagar debe mostrar “SALDO PENDIENTE” ya que sin los minutos es imposible sacar el saldo pendiente. Debe ordenar por saldo a pagar descendente.

select
par.equipo_local as "EQUIPO LOCAL",
par.equipo_visita as "EQUIPO VISITA",
case
when pro.minutos_jugados is null then 'NO INFORMADOS'
else pro.minutos_jugados::text
end as "MINUTOS JUGADOS",
to_char(tar.valor,'$99,999') as "TARIFA APLICADA",
case
when pro.minutos_jugados is null then 'SALDO PENDIENTE'
else to_char((pro.minutos_jugados::decimal/60)*tar.valor,'$999,999')
end as "TOTAL PARCIAL",
to_char(pro.valor_pagado,'$99,999') as "ABONADO",
case
when pro.minutos_jugados is null then 'SALDO PENDIENTE'
else to_char(((pro.minutos_jugados::decimal/60)*tar.valor)-pro.valor_pagado,'$999,999')
end as "SALDO A PAGAR"
from partido par
inner join programacion pro on pro.id_programacion = par.id_programacion
inner join tarifa tar on tar.id_tarifa = par.id_tarifa
order by "SALDO A PAGAR" desc;

<p align="center">
  <img src="https://raw.githubusercontent.com/matiassanmartins/SQL/refs/heads/main/CONSULTA/CASO_CAMPUS/IMAGENES/RESPUESTA6.PNG" title="hover text">
</p>