bd_shortcode = function(code) htmltools::HTML(code)
bd_ref = function(filename) {
  filename = sub("Rmd$", "html", filename)
  bd_shortcode(paste0('{{< relref "', filename, '" >}}'))
}
bd_relref = function(filename) {
  filename = sub("Rmd$", "html", filename)
  htmltools::HTML(paste0('{{< ref "', filename, '" >}}'))
}