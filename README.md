# tfg-IRPF-ComunidadMadrid-s80-20
# Impacto de políticas fiscales sobre índices de desigualdad en la Comunidad de Madrid

# Este repositorio contiene el código y las interpretaciones de un análisis realizado mediante un Diff-diff en el que se mide como el índice de desigualdad s80/20 se ve modificado por políticas de rebaja fiscal en el IRPF, comparando con otras comunidades en el mismo tiempo con otras comunidades autónomas que no realizaron rebajas fiscales.

# Herramientas utilizadas
  # Bases de datos obtenidas de AEAT e INE
  # Software de análisis: Stata
  # Modelo econométrico: Diferencias en diferencias

# Código utilizado para el análisis de datos 
# //empezamos el análisis con todas las variables//
import excel "C:\Users\aulavirtual\Downloads\tfg\Datos_organizados.xlsx", sheet("datos organizados") firstrow clear
rename Año year
rename ComunidadAutónoma community
rename Decil decil
rename Tipomedio tipo_med
rename Númerodedeclarantes num_decl
rename Rendimientosdeltrabajoeuros rend_trab
rename S80S20 s80s20
rename Rendimientosdelcapitalmobilia rend_cap_mob
rename Rendimientosdelcapitalinmobil rend_cap_inm
rename PIBpc pib_pc
rename Rendimientostotaleseuros rend_tot
# //todas las variables con nombre de código e instalamos diferentes paquetes para las salidas//
ssc install estout
ssc install outreg2 
ssc install asdoc
# //esttab, star(* 0.05 ** 0.01 *** 0.001)
# //corregimos community para que se pueda leer//
encode community, gen(com_id)
# //empezamos con la primera regresión normal con mas controles//
# //Primero va a ser sin crear las variables de rendimientos de capital, luego se sumarán//
regress s80s20 tipo_med pib_pc u
# //OLS efectos fijos con clm como comundad autonoma de referencia//
regress s80s20 tipo_med pib_pc u ib14.com_id i.year
regress s80s20 tipo_med pib_pc u ib14.com_id i.year, robust
# //para exportar//
asdoc regress s80s20 tipo_med pib_pc u ib6.com_id i.year, robust, save(result_regr.doc),replace
# //vamos a incluir las nuevas variables//
gen rend_trab_pc=rend_trab/num_decl
gen rend_cap=rend_cap_mob+rend_cap_inm
gen rend_cap_pc=rend_cap/num_decl
gen pct_rend_lab=(rend_trab/rend_tot)*100
gen pct_rend_cap=(rend_cap/rend_tot)*100
label variable rend_trab_pc "rendimientos del trabajo per capita"
label variable rend_cap "rendimientos capital totales"
label variable rend_cap_pc "rendimientos capital per capita"
label variable pct_rend_lab "% rendimientos trabajo"
label variable pct_rend_cap "% rendimientos capital"
# //hacemos la regresión con las nuevas variables//
regress s80s20 tipo_med rend_trab_pc rend_cap_pc pib_pc u ib6.com_id i.year, robust
# //deja de ser significativo tipo_medio//
# //deciles igual están truncando el estudio, probamos a usarlo de efecto fijo//
encode decil, gen(decil_id)
# //hacemos regresión con deciles como efecto fijo//
regress s80s20 tipo_med rend_cap_pc rend_trab_pc u pib_pc i.decil_id ib14.com_id##i.year,robust
regress s80s20 tipo_med rend_cap_pc rend_trab_pc u pib_pc i.decil_id ib14.com_id i.year, robust
regress s80s20 tipo_med rend_cap_pc rend_trab_pc u pib_pc i.decil_id i.year, robust
# //lo guardamos para interpretarlo y pasarselo al profe, ENTREGADO EL 10 de junio//
asdoc regress s80s20 u pib_pc tipo_med rend_cap_pc rend_trab_pc, robust save(Regresiones.doc) replace
asdoc regress s80s20 u pib_pc tipo_med rend_cap_pc rend_trab_pc ib6.com_id i.year, robust save(Regresiones.doc) append
asdoc regress s80s20 u pib_pc tipo_med rend_cap_pc rend_trab_pc ib6.com_id i.decil_id i.year, robust save(Regresiones.doc) append
asdoc regress s80s20 u pib_pc tipo_med rend_cap_pc rend_trab_pc i.decil_id i.year, robust save(Regresiones.doc) append
asdoc regress s80s20 u pib_pc tipo_med rend_cap_pc rend_trab_pc ib6.com_id i.decil_id, robust save(Regresiones.doc) append

# ///primer diff-diff escogiendo Comunidad de Madrid como tratamiento y sin escoger nignún grupo de control(todos los que no son com mad)//
gen treatment=(com_id==14)
gen post=(year>=2008)
regress s80s20 i.treatment##i.post tipo_med rend_cap_pc rend_trab_pc u pib_pc i.decil_id, robust
# //el resultado del diff-diff con tanta heterogeneidad en el grupo de control no ha proporcionado datos significativos por lo que ahora seleccionamos tres comunidades autónomas que son Cataluña, Aragón y la rioja//
keep if inlist(com_id, 2,9,14,11,13,8)
regress s80s20 i.treatment##i.post rend_cap_pc rend_trab_pc u pib_pc i.decil_id, robust

# //comprobación tendencias paralelas//
regress s80s20 i.treatment##ib2007.year if year <= 2007, robust
 test 1.treatment#2003.year 1.treatment#2004.year 1.treatment#2005.year 1.treatment#2006.year
 # //exportamos//
 asdoc regress s80s20 i.treatment##ib2007.year if year <= 2007, robust replace save(resultados_tendencias_paralel.doc)
  asdoc test 1.treatment#2003.year 1.treatment#2004.year 1.treatment#2005.year 1.treatment#2006.year, append save(resultados_tendencias_paralel.doc)
  
  # //hacemos regresión con el grupo de control y con efectos fijos//
  regress s80s20 i.treatment##i.post rend_cap_pc u pib_pc i.com_id i.year
  # //hacemos un test de colinealidad y sale muy alto para todos los controles igual que también sale muy alto para algunas de las comunidades autonomas//
 # // exportamos la regresión//
 asdoc   regress s80s20 i.treatment##i.post rend_cap_pc u pib_pc i.com_id i.year, robust replace save(diff-diff_FE_controls.doc)
 
 # /// regresión final con efectos fijos de año//
regress s80s20 i.treatment##i.year rend_cap_pc u pib_pc i.com_id i.decil_id,robust

regress s80s20 i.treatment##ib2008.year rend_cap_pc pib_pc i.com_id i.year,robust
# //encontramos que no tiene sentido usar efectos fijos de decil//
keep if decil_id=="Total"
# //creamos pib_pc_sq//
sum pib_pc, meanonly
scalar mean_pib=r(mean)
gen pib_pc_centered = pib_pc - mean_pib
gen pib_pc_sq = pib_pc_centered^2

# //regresión final//
regress s80s20 i.treatment##i.year rend_cap_pc pib_pc_sq i.com_id i.year


# Interpretaciones finales del estudio
# Se puede interpretar de los resultados obtenidos que hay un incremento significativo de dicho índice de desigualdad causado por la rebaja fiscal llevada a cabo por la comunidad de Madrid en 2007. La literatura ofrece diferentes motivos para este suceso como puede ser el efecto llamada para grandes rentas o el notorio crecimiento de rentas de capital, cuyos principales dueños pertenecen al quintil superior. Este proyecto pretende ofrecer una base metodológica con la que estudiar la relación de las políticas fiscales del IRPF y su influencia en los índices de  desigualdad. 

