# pruebaShinyMapas

Prueba de shiny dashboard con mapa y un selector de capas en el panel izquierdo.



library(shinydashboard)
library(leaflet)
library(ggplot2)


source("load_data.R", local = TRUE)

espacios <- st_read(dsn ="~/Documentos/R/Datos/datos.gpkg" ,layer = "espacios")
densidadPoblacional <- st_read(dsn ="~/Documentos/Data Cultura/Monitor/datos/barrios.gpkg")

todos <- c("CENTRO CULTURAL","BAR","BIBLIOTECA")
# -----------------------------------------------------------------------------
# Dashboard UI
# -----------------------------------------------------------------------------
ui <- dashboardPage(
  dashboardHeader(
    title = "Espacios Culturales"
  ),
  dashboardSidebar(
    sidebarMenu(
      menuItem("Tab 1", tabName = "tabResumen", icon = icon("database"))
    ),
    div(
      numericInput("valor",
                   "Minimum Population (in millions)",
                   min = 1,
                   max = 3500,
                   value = 3000)
    
    ),
    div(
      radioButtons("radio", label = h3("Radio buttons"),
                   choices = list("cc" = 'CENTRO CULTURAL', "bar" = 'BAR', "todos" = todos), 
                   selected = 1)
    )
   
  ),
  dashboardBody(
    tabItems(
     
      # Resumen ------------------------------------------------------
      tabItem(tabName = "tabResumen",
              
                fluidRow(
                  box(width = 12, title = "Espacios Culturales en CABA",
                      leafletOutput("current_map", height = 800)
                  )
                ))    )))
    




# -----------------------------------------------------------------------------
# Dashboard server code
# -----------------------------------------------------------------------------
server <- function(input, output) {
  
  
  espacios_data <- reactive({
    espacios %>% filter(id < input$valor) %>% filter(FUNCIÓN.PRINCIPAL %in% input$radio) 
      })
  
  
  output$current_map <- renderLeaflet({
    
    
leaflet() %>% 
      addProviderTiles(providers$Stamen.Toner) %>% 
      setView(lng= -58.436, lat=-34.604, zoom=15) %>% 
      #addPolygons(data= barrios,
      #           color="#660000",
      #          weight = 1,
      #         smoothFactor = 0.5) %>%  #smooth las lineas para que el mapa cargue mas rapido 
      addCircleMarkers(lng = espacios$LONGITUD, 
                       lat = espacios$LATITUD,
                       color = '#000000',
                           radius=5)
  })
  
  observe({
  leafletProxy("current_map", data=espacios_data()

               ) %>% 
      clearMarkers() %>% 
      addCircleMarkers(lng = espacios_data()$LONGITUD, 
                       lat = espacios_data()$LATITUD,
                       color = '#000000',
                       radius=5)
}  )
}

shinyApp(ui, server)
