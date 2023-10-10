# nginx.conf_DASH
Archivo nginx.conf con DASH activado


    user www-data;
    worker_processes auto;
    pid /run/nginx.pid;
    include /etc/nginx/modules-enabled/*.conf;

    events {
        worker_connections  1024;
    }

## Configuración de RTMP


    rtmp {
    server {
        listen 1935; # Escuchar en el puerto estándar de RTMP
        chunk_size 4000;
        application show {
            live on;
            # Activar el protocolo HLS
            hls on;
            hls_path /var/www/hls;
            hls_fragment 3;
            hls_playlist_length 60;
            # Inhabilitar el streaming desde nginx usando rtmp
            dash on;
	    dash_path /var/www/dash;
            dash_fragment 5s;
            deny play all;
        }

        # Permitir reproducir las grabaciones de los streams en directo
        # usando una dirección URL como la siguiente:
        # "rtmp://ip_servidor:1935/vod/nombre_video.mp4"
        application vod {
            play /var/www/videos/;
        }

    }
}

## Configuración de HTTP



        http {
          sendfile off;
          tcp_nopush on;
          directio 512;
          default_type application/octet-stream;
            include mime.types;
         server {
        listen 80;

        location / {
            # Deshabilitar la caché
            add_header 'Cache-Control' 'no-cache';

            # Configuración de CORS
            add_header 'Access-Control-Allow-Origin' '*' always;
            add_header 'Access-Control-Expose-Headers' 'Content-Length';

            # Permitir solicitudes CORS preflight
            if ($request_method = 'OPTIONS') {
                add_header 'Access-Control-Allow-Origin' '*';
                add_header 'Access-Control-Max-Age' 1728000;
                add_header 'Content-Type' 'text/plain charset=UTF-8';
                add_header 'Content-Length' 0;
                return 204;
            }

            types {
                text/html html;
                application/x-shockwave-flash swf;
                text/css css;
                application/x-javascript js;
                application/dash+xml mpd;               
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
                video/mp4 mp4;
            }

            root /var/www/;
            index  index.html index.htm;
        }

        # Esta URL proporciona estadísticas de RTMP en formato XML
        location /stat {
            rtmp_stat all;
            rtmp_stat_stylesheet stat.xsl;
        }
 
        location /stat.xsl {
            # Hoja de estilo XML para ver las estadísticas de RTMP
            # El archivo stat.xsl deberá estar en la ubicación siguiente:
            root /var/www/;
        }

	  # Permite ver estadísticas sobre el estado de los espectadores en
  ## Nginx usando la URL: "http://ip_servidor/stats"     
        location /stats {
            stub_status;
        }
    }
}
