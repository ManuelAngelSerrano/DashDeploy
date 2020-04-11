# Deploy una aplicación Flask en Ubuntu
## Instalación de aplicaciones
1. sudo apt install python3 python3-dev
2. sudo apt install python3-pip
3. sudo apt install apache2
4. sudo apt install libapache2-mod-wsgi-py3
5. sudo pip3 install dash
6. sudo pip3 install dash-bootstrap-components

## Configuracion del sitio web
**Nota:** *NombreApp*=basic

1. Crear /var/www/basic
2. Cambiar propietario carpeta:
    ```sh
    sudo chown -R $USER:$USER /var/www/basic
    ```
3. Cambiar permisos a /var/www
    ```sh
    sudo chmod 755 /var/www
    ```
4. crear un virtualenv e instalar las librerias de dash
    ```sh
    python3 -m venv /var/www/basic/venv
    source /var/www/basic/venv/bin/activate
    pip3 install dash dash-bootstrap-components
    deactivate
    ```
5. Crear /var/www/basic/basic.wsgi
6. poner en el archivo .wsgi:
    ```python
    #! /usr/bin/python3
    import sys
    
    sys.path.insert(0, "/var/www/basic/")
    
    from basic import server as application
    ```
7. copiar aplicacion a /var/www/basic
    **Nota: Hay que añadir:** 
    ```python
    server = app.server
    ```
    despues de:
    ```python
    app = dash.Dash(__name__, external_stylesheets=[dbc.themes.BOOTSTRAP])
    ```
    
    y quitar las 2 últimas lineas que son para ejecutar el servidor de pruebas (*puede no ser necesario*):
    
    ```python
    #if __name__ == "__main__":
    #    app.run_server()
    ```
    
    **basic.py**
    ```python
    import dash
    import dash_bootstrap_components as dbc
    import dash_core_components as dcc
    import dash_html_components as html
    
    navbar = dbc.NavbarSimple(
        children=[
            dbc.NavItem(dbc.NavLink("Link", href="#")),
            dbc.DropdownMenu(
                nav=True,
                in_navbar=True,
                label="Menu",
                children=[
                    dbc.DropdownMenuItem("Entry 1"),
                    dbc.DropdownMenuItem("Entry 2"),
                    dbc.DropdownMenuItem(divider=True),
                    dbc.DropdownMenuItem("Entry 3"),
                ],
            ),
        ],
        brand="Demo",
        brand_href="#",
        sticky="top",
    )
    
    body = dbc.Container(
        [
            dbc.Row(
                [
                    dbc.Col(
                        [
                            html.H2("Heading"),
                            html.P(
                                """\
    Donec id elit non mi porta gravida at eget metus.
    Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum
    nibh, ut fermentum massa justo sit amet risus. Etiam porta sem
    malesuada magna mollis euismod. Donec sed odio dui. Donec id elit non
    mi porta gravida at eget metus. Fusce dapibus, tellus ac cursus
    commodo, tortor mauris condimentum nibh, ut fermentum massa justo sit
    amet risus. Etiam porta sem malesuada magna mollis euismod. Donec sed
    odio dui."""
                            ),
                            dbc.Button("View details", color="secondary"),
                        ],
                        md=4,
                    ),
                    dbc.Col(
                        [
                            html.H2("Graph"),
                            dcc.Graph(
                                figure={"data": [{"x": [1, 2, 3], "y": [1, 4, 9]}]}
                            ),
                        ]
                    ),
                ]
            )
        ],
        className="mt-4",
    )
    
    app = dash.Dash(__name__, external_stylesheets=[dbc.themes.BOOTSTRAP])
    server = app.server
    
    app.layout = html.Div([navbar, body])
    
    #if __name__ == "__main__":
    #    app.run_server()
    ```
8. touch /var/www/basic/__init__.py
9. sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/basic.conf
    ~~~
    <VirtualHost *:80>
    
	    ErrorLog ${APACHE_LOG_DIR}/error.log
	    CustomLog ${APACHE_LOG_DIR}/access.log combined
    
	    ServerName localhost
    
	    WSGIDaemonProcess basic user=manolo group=users threads=5
	    WSGIScriptAlias / /var/www/basic/basic.wsgi
    
	    <Directory /var/www/basic>
		    WSGIProcessGroup basic
		    WSGIApplicationGroup %{GLOBAL}
		    #Order deny,allow
		    #Allow from all
		    Require all granted
	    </Directory>
    </VirtualHost>
    ~~~
10. activar wsgi: 
    ~~~
    sudo a2enmod wsgi
    ~~~
11. activar el nuevo sitio:
    ~~~
    sudo a2ensite basic.conf
    ~~~
12. desactivar el sitio por defecto de apache:
    ~~~
    sudo a2dissite 000-default.conf
    ~~~
13. reiniciar el servidor apache: 
    ~~~
    sudo systemctl restart apach2.service
    ~~~
