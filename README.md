## Deploy Grafana-Prometheus-cAdvisor

## 1.Giới thiệu.

    - Grafana là công cụ hiện thị dữ liệu trực quan cho phép chọn nguồn dữ liệu để hiển thị(ở đây chọn prometheus).

    - Prometheus chủ động pull dữ liệu về(metrics) được export từ các Exporter.

    - cAdvisor(được coi như là Exporter) export các metrics của các docker service, các process trên server(Exporter).
      Có nhiều Exporter khác như: Node Exporter, Postgres Exporter....

## 2. Deploy
   
   ### Bước 1: Deploy sử dụng docker command.
 
       - docker-compose -f grafana.docker-compose.yml up -d

       - Notes : + Trong file grafana.docker-compose.yml có cài luôn cAdvisor để monitor luôn chính server cài grafana.
                 + Nếu k cần cài cAdvisor trên server cài grafana và prometheus thì xóa đoạn sau đi:
                 #Install cadvisor
                  cadvisor:
                    image: gcr.io/google-containers/cadvisor:latest
                    container_name: cadvisor
                    volumes:
                       - /:/rootfs:ro
                       - /var/run:/var/run:rw
                       - /sys:/sys:ro
                       - /var/lib/docker/:/var/lib/docker:ro
                    devices:
                       - /dev/kmsg:/dev/kmsg
                    ports:
                       - 8088:8080"
                  + Nếu không cần monitor thêm server nào khác server cài grafana thì không cần thực hiện Bước 2.
              
                   
   ### Bước 2: Deploy cAdvisor lên các server cần theo dõi docker.
      - docker-compose -f cadvisor.docker-compose.yml up -d

   ### Bước 3 : Sửa config trong file prometheus.yml để báo cho prometheus biết nó cần pull dữ liệu từ những server nào
        - job_name: cadvisor
          scrape_interval: 10s
          static_configs:
            - targets: ['ip:8080','ip:8088'] # ip và port của Exporter cAdvisor
   ### Sau khi install xong thì vào trang : http://localhost:3000/ để xem kết quả
