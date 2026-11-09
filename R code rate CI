# RATE (CI 95%)
# Fuente unica: Temperatura intervalos.xlsx, hoja general area.
# Se leen Rate, LL y UL tal como estan guardados en ese Excel.
# No se calculan tasas, intervalos, denominadores ni datos nuevos.
# No se escribe en el Excel. Las celdas vacias permanecen como NA.
# La tabla de revision conserva los 33 departamentos/distrito.
# En cada figura se incluyen solo filas con Rate, LL y UL completos.
# El formato numerico de dos decimales solo afecta a las etiquetas.
# Se ordena por tasa descendente; en empates, por codigo ISO.
# Codigos contrastados con UNECE:
# https://vocabulary.uncefact.org/unlocode-subdivisions

# CONFIGURACION: carpeta y nombre del Excel de la ultima captura.
carpeta <- "C:/Users/ACER/Desktop/Codigo para R IC"
nombre_excel <- "Temperatura intervalos.xlsx"
hoja_excel <- "general area"

# Los titulos reproducen literalmente los seis nombres de la referencia.
# F representa temperature del grupo md, aunque su titulo sea Physician visits.
series <- data.frame(
  letra = LETTERS[1:6],
  titulo = c("Overall heat visits", "Overall cold visits", "Overall temperature",
             "Physician heat visits", "Physician cold visits", "Physician visits"),
  eje_x = c("Overall heat visits (95% CI)", "Overall cold visits (95% CI)",
            "Overall temperature visits (95% CI)", "Physician heat visits (95% CI)",
            "Physician cold visits (95% CI)", "Physician temperature visits (95% CI)"),
  tasa = c("Rate_overall_heat", "Rate_overall_cold", "Rate_overall_temperature",
           "Rate_md_heat", "Rate_md_cold", "Rate_md_temperature"),
  inferior = c("LL_overall_heat", "LL_overall_cold", "LL_overall_temperature",
               "LL_md_heat", "LL_md_cold", "LL_md_temperature"),
  superior = c("UL_overall_heat", "UL_overall_cold", "UL_overall_temperature",
               "UL_md_heat", "UL_md_cold", "UL_md_temperature"),
  stringsAsFactors = FALSE
)

detener <- function(...) stop(paste0(...), call. = FALSE)

preparar_readxl <- function() {
  if (!requireNamespace("readxl", quietly = TRUE)) {
    message("Instalando readxl. Esta primera instalacion requiere Internet.")
    utils::install.packages("readxl", repos = "https://cloud.r-project.org")
  }
  if (!requireNamespace("readxl", quietly = TRUE)) {
    detener("Falta readxl. Instalelo desde Tools > Install Packages y ejecute de nuevo.")
  }
}

clave_departamento <- function(x) {
  # Se usa una clave auxiliar solo para emparejar nombres con sus codigos.
  # La columna Department original se conserva para la auditoria.
  y <- iconv(x, from = "UTF-8", to = "ASCII//TRANSLIT")
  if (anyNA(y)) detener("No se pudieron interpretar todos los nombres de departamentos.")
  tolower(trimws(y))
}

diccionario_iso <- function() {
  stats::setNames(c(
    "CO-AMA", "CO-ANT", "CO-ARA", "CO-SAP", "CO-ATL", "CO-DC", "CO-BOL", "CO-BOY",
    "CO-CAL", "CO-CAQ", "CO-CAS", "CO-CAU", "CO-CES", "CO-CHO", "CO-CUN", "CO-COR",
    "CO-GUA", "CO-GUV", "CO-HUI", "CO-LAG", "CO-MAG", "CO-MET", "CO-NAR", "CO-NSA",
    "CO-PUT", "CO-QUI", "CO-RIS", "CO-SAN", "CO-SUC", "CO-TOL", "CO-VAC", "CO-VAU", "CO-VID"
  ), c(
    "amazonas", "antioquia", "arauca", "san andres", "atlantico", "bogota", "bolivar", "boyaca",
    "caldas", "caqueta", "casanare", "cauca", "cesar", "choco", "cundinamarca", "cordoba",
    "guainia", "guaviare", "huila", "la guajira", "magdalena", "meta", "narino", "norte de santander",
    "putumayo", "quindio", "risaralda", "santander", "sucre", "tolima", "valle del cauca", "vaupes", "vichada"
  ))
}

leer_y_validar <- function(archivo, hoja) {
  if (!file.exists(archivo)) {
    detener("No se encuentra: ", archivo,
            ". Coloque el Excel indicado en la carpeta o ajuste nombre_excel/carpeta.")
  }
  preparar_readxl()
  if (!hoja %in% readxl::excel_sheets(archivo)) detener("No existe la hoja '", hoja, "'.")
  cabecera <- readxl::read_excel(archivo, sheet = hoja, n_max = 0, .name_repair = "minimal")
  requeridas <- c("Department", as.vector(t(as.matrix(series[, c("tasa", "inferior", "superior")]))))
  if (anyDuplicated(names(cabecera)) || !setequal(names(cabecera), requeridas) ||
      names(cabecera)[1] != "Department") {
    detener("Las columnas no corresponden al Excel combinado de 19 columnas examinado. ",
            "No se usaran los archivos overall/physician separados.")
  }
  datos <- readxl::read_excel(
    archivo, sheet = hoja, col_types = c("text", rep("numeric", 18)),
    trim_ws = FALSE, .name_repair = "minimal"
  )
  if (nrow(datos) != 33L || anyNA(datos$Department) || any(!nzchar(trimws(datos$Department)))) {
    detener("Se requieren los 33 departamentos/distrito, con nombre en cada fila.")
  }
  claves <- clave_departamento(datos$Department)
  if (anyDuplicated(claves)) detener("Hay departamentos repetidos. No se sumaran ni eliminaran filas.")
  iso <- unname(diccionario_iso()[claves])
  if (anyNA(iso) || anyDuplicated(iso)) detener("Hay un departamento sin codigo ISO unico.")
  for (i in seq_len(nrow(series))) {
    cols <- unlist(series[i, c("tasa", "inferior", "superior")], use.names = FALSE)
    m <- as.matrix(datos[, cols])
    cuantos_na <- rowSums(is.na(m))
    parcial <- which(!cuantos_na %in% c(0L, 3L))
    if (length(parcial)) {
      detener("Estimacion parcialmente vacia en ", series$titulo[i], ": ",
              paste(datos$Department[parcial], collapse = ", "), ".")
    }
    disponibles <- cuantos_na == 0L
    if (!any(disponibles)) detener("No hay estimaciones disponibles para ", series$titulo[i], ".")
    if (any(!is.finite(m[disponibles, , drop = FALSE])) || any(m[disponibles, , drop = FALSE] < 0)) {
      detener("Hay valores no finitos o negativos en ", series$titulo[i], ".")
    }
    invalidos <- which(disponibles & (m[, 2] > m[, 1] | m[, 1] > m[, 3]))
    if (length(invalidos)) {
      detener("La tasa queda fuera de sus limites en ", series$titulo[i], ": ",
              paste(datos$Department[invalidos], collapse = ", "), ".")
    }
  }
  list(datos = datos, iso = iso)
}

preparar_serie <- function(datos, iso, especificacion) {
  tasa <- datos[[especificacion$tasa]]
  disponibles <- which(!is.na(tasa))
  vacios <- which(is.na(tasa))
  orden <- c(disponibles[order(-tasa[disponibles], iso[disponibles], method = "radix")],
             vacios[order(iso[vacios], method = "radix")])
  data.frame(
    figura = especificacion$letra, titulo = especificacion$titulo,
    fila_excel = orden + 1L, Department = datos$Department[orden], ISO = iso[orden],
    columna_tasa = especificacion$tasa, columna_LL = especificacion$inferior,
    columna_UL = especificacion$superior,
    Rate = datos[[especificacion$tasa]][orden],
    LL = datos[[especificacion$inferior]][orden],
    UL = datos[[especificacion$superior]][orden],
    incluida_en_figura = !is.na(tasa[orden]),
    stringsAsFactors = FALSE
  )
}

dibujar_intervalos <- function(d, especificacion, panel = FALSE) {
  # Seleccion para el dibujo, sin modificar la tabla completa de revision.
  d <- d[stats::complete.cases(d[, c("Rate", "LL", "UL")]), , drop = FALSE]
  n <- nrow(d)
  y <- n:1
  etiquetas <- sprintf("%.2f (%.2f - %.2f)", d$Rate, d$LL, d$UL)
  maximo <- max(d$UL)
  if (maximo == 0) maximo <- 1
  graphics::plot.new()
  cex_valores <- if (panel) 0.57 else 0.65
  ancho_etiquetas <- max(graphics::strwidth(etiquetas, units = "inches", cex = cex_valores))
  fraccion <- ancho_etiquetas / graphics::par("pin")[1]
  if (fraccion >= 0.6) detener("La figura es demasiado estrecha para mostrar las etiquetas completas.")
  espacio <- maximo * 0.025
  xmax <- (maximo + espacio) / (1 - fraccion) * 1.05
  graphics::plot.window(xlim = c(0, xmax), ylim = c(0.5, n + 0.5), xaxs = "i", yaxs = "i")
  graphics::rect(0, 0.5, xmax, n + 0.5, col = "#D5E2EE", border = NA)
  graphics::abline(h = seq(0.5, n + 0.5, by = 1), col = "white", lwd = 0.8)
  # Las tres coordenadas numericas salen directamente de Rate, LL y UL.
  graphics::rect(0, y - 0.31, d$Rate, y + 0.31,
                 col = "#16344E", border = NA)
  graphics::segments(d$LL, y, d$UL, y, col = "#D85B67", lwd = 0.9)
  graphics::segments(d$LL, y - 0.11, d$LL, y + 0.11,
                     col = "#D85B67", lwd = 0.9)
  graphics::segments(d$UL, y - 0.11, d$UL, y + 0.11,
                     col = "#D85B67", lwd = 0.9)
  graphics::text(d$UL + espacio, y, etiquetas, adj = c(0, 0.5),
                 cex = cex_valores, col = "#111111")
  ticks <- pretty(c(0, xmax), n = 5)
  ticks <- ticks[ticks >= 0 & ticks <= xmax]
  graphics::axis(1, at = ticks, cex.axis = 0.72, tck = -0.012,
                 col = "#617181", col.axis = "#16212B")
  graphics::axis(2, at = y, labels = d$ISO, las = 1, tick = FALSE,
                 cex.axis = if (panel) 0.64 else 0.74, font = 2, line = -0.25)
  graphics::mtext(especificacion$eje_x, side = 1, line = 2.1, cex = 0.78, font = 2)
  graphics::mtext("Colombian departments (ISO 3166-2)", side = 2, line = 3.5,
                  cex = 0.75, font = 2)
  graphics::title(main = especificacion$titulo, font.main = 2,
                  cex.main = if (panel) 1.45 else 1.55, line = 0.9)
  graphics::mtext(especificacion$letra, side = 3, line = 0.8, adj = -0.13,
                  cex = 1.35, font = 2)
  graphics::mtext("2010-2024 visits per 10,000 inhabitants (DANE 2024).",
                  side = 1, line = 3.4, cex = 0.57, col = "#444444")
  invisible(NULL)
}

guardar_figura <- function(ruta, formato, dibujar, panel = FALSE) {
  ancho <- if (panel) 24 else 8
  alto <- if (panel) 16 else 8
  if (formato == "pdf") {
    grDevices::pdf(ruta, width = ancho, height = alto, family = "serif",
                   pointsize = 12, bg = "white", useDingbats = FALSE)
  } else {
    args <- list(filename = ruta, width = ancho * 300, height = alto * 300,
                 res = 300, pointsize = 12, bg = "white")
    if (isTRUE(capabilities("cairo"))) args$type <- "cairo"
    do.call(grDevices::png, args)
  }
  dispositivo <- grDevices::dev.cur()
  on.exit(grDevices::dev.off(which = dispositivo), add = TRUE)
  graphics::par(mfrow = if (panel) c(2, 3) else c(1, 1), cex = 1,
                 mar = c(5, 5.1, 3.6, 1.3), family = "serif", xpd = FALSE)
  dibujar()
  invisible(ruta)
}

generar_intervalos <- function(carpeta, nombre_excel, hoja_excel = "general area") {
  archivo <- file.path(carpeta, nombre_excel)
  if (!file.exists(archivo)) detener("No se encuentra el Excel: ", archivo, ".")
  huella <- unname(tools::md5sum(archivo))
  if (is.na(huella)) detener("No se pudo obtener la huella del Excel.")
  on.exit({
    if (!identical(huella, unname(tools::md5sum(archivo)))) {
      warning("El Excel cambio durante la ejecucion. Revise si se guardo desde otra aplicacion.")
    }
  }, add = TRUE)
  origen <- leer_y_validar(archivo, hoja_excel)
  tablas <- lapply(seq_len(nrow(series)), function(i) {
    preparar_serie(origen$datos, origen$iso, series[i, ])
  })
  # Comprueba los valores despues de ordenar, incluidos los NA originales.
  for (i in seq_along(tablas)) {
    d <- tablas[[i]]
    fila <- d$fila_excel - 1L
    if (!identical(d$Rate, origen$datos[[series$tasa[i]]][fila]) ||
        !identical(d$LL, origen$datos[[series$inferior[i]]][fila]) ||
        !identical(d$UL, origen$datos[[series$superior[i]]][fila])) {
      detener("Los valores ordenados no coinciden con el Excel en la figura ", series$letra[i], ".")
    }
  }
  raiz <- file.path(carpeta, "resultados_intervalos")
  if (!dir.exists(raiz) && !dir.create(raiz, recursive = TRUE)) detener("No se pudo crear la carpeta de resultados.")
  sello <- format(Sys.time(), "%Y%m%d_%H%M%S")
  salida <- file.path(raiz, sello)
  k <- 1L
  while (file.exists(salida) || dir.exists(salida)) {
    k <- k + 1L
    salida <- file.path(raiz, paste0(sello, "_", k))
  }
  if (!dir.create(salida)) detener("No se pudo crear: ", salida)
  for (i in seq_along(tablas)) {
    message("Generando ", series$letra[i], ": ", series$titulo[i], " ...")
    for (formato in c("pdf", "png")) {
      guardar_figura(file.path(salida, paste0(series$titulo[i], ".", formato)), formato,
        dibujar = function() dibujar_intervalos(tablas[[i]], series[i, ]))
    }
  }
  for (formato in c("pdf", "png")) {
    guardar_figura(file.path(salida, paste0("Panel 6 figuras.", formato)), formato,
      panel = TRUE, dibujar = function() {
        for (i in seq_along(tablas)) dibujar_intervalos(tablas[[i]], series[i, ], panel = TRUE)
      })
  }
  auditoria <- do.call(rbind, tablas)
  utils::write.csv2(auditoria, file.path(salida, "auditoria_valores.csv"),
                    row.names = FALSE, na = "", fileEncoding = "UTF-8")
  resumen <- do.call(rbind, lapply(seq_along(tablas), function(i) {
    data.frame(figura = series$letra[i], titulo = series$titulo[i],
      filas_fuente = nrow(tablas[[i]]), filas_en_figura = sum(tablas[[i]]$incluida_en_figura),
      vacias_en_fuente = sum(is.na(tablas[[i]]$Rate)), valores_preservados = TRUE,
      stringsAsFactors = FALSE)
  }))
  utils::write.csv2(resumen, file.path(salida, "control_figuras.csv"),
                    row.names = FALSE, fileEncoding = "UTF-8")
  if (!identical(huella, unname(tools::md5sum(archivo)))) {
    detener("El archivo fuente cambio durante la ejecucion. Revise los archivos antes de usar las figuras.")
  }
  notas <- c(
    paste("Excel fuente:", normalizePath(archivo, winslash = "/", mustWork = TRUE)),
    paste("Hoja:", hoja_excel), paste("Huella MD5 antes y despues:", huella),
    "Se grafico directamente cada Rate, LL y UL. No se recalculo ningun valor.",
    "Los numeros de las etiquetas tienen dos decimales; las coordenadas usan los valores leidos.",
    "Las barras azules representan Rate. Las lineas rojas y sus extremos representan LL y UL.",
    "Cada figura se ordena por tasa de mayor a menor. En empate, por codigo ISO.",
    "La tabla de revision conserva las 33 filas originales de cada serie.",
    "Cada figura dibuja solo las filas con Rate, LL y UL completos; las demas conservan sus celdas vacias en la tabla de revision.",
    "No se imputaron ceros ni intervalos. incluida_en_figura identifica las filas dibujadas.",
    "Cada figura usa un eje ajustado a sus valores; consulte sus marcas antes de comparar longitudes entre figuras.",
    "La interpretacion del archivo examinado es: atenciones 2010-2024 / poblacion DANE 2024 x 10.000.",
    "El script no audita nuevamente la descarga SISPRO ni reconstruye numeradores o denominadores.",
    "F se titula Physician visits como en la referencia y usa las columnas md_temperature.",
    "Codigos: https://vocabulary.uncefact.org/unlocode-subdivisions",
    "auditoria_valores.csv conserva departamento, fila y columnas fuente, valores e identificacion de figura.",
    "", capture.output(utils::sessionInfo())
  )
  writeLines(enc2utf8(notas), file.path(salida, "LEEME.txt"), useBytes = TRUE)
  message("Listo. Seis figuras individuales y un panel, en PDF y PNG a 300 ppp.")
  message("El Excel original conserva su huella. No se modificaron sus datos.")
  message("Resultados: ", normalizePath(salida, winslash = "/", mustWork = TRUE))
  print(resumen, row.names = FALSE)
  invisible(salida)
}

# EJECUCION COMPLETA.
resultado <- generar_intervalos(carpeta, nombre_excel, hoja_excel)
