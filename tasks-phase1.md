IMPORTANT ❗ ❗ ❗ Please remember to destroy all the resources after each work session. You can recreate infrastructure by creating new PR and merging it to master.
  
![img.png](doc/figures/destroy.png)

1. Authors:

   13

   https://github.com/TheStm/tbd-workshop-1
   
2. Follow all steps in README.md.

3. In boostrap/variables.tf add your emails to variable "budget_channels".

4. From avaialble Github Actions select and run destroy on main branch.
   
5. Create new git branch and:
    1. Modify tasks-phase1.md file.
    
    2. Create PR from this branch to **YOUR** master and merge it to make new release. 
    
    ![release.png](doc/figures/release.png)


6. Analyze terraform code. Play with terraform plan, terraform graph to investigate different modules.

   W ramach analizy wybrany został moduł `vpc`, odpowiedzialny za utworzenie podstawowej infrastruktury sieciowej w 
   projekcie GCP.

   Zasoby tworzone przez ten moduł to:
   - `google_compute_network.network` – główna sieć VPC.
   - `google_compute_subnetwork.subnetwork` – jedna lub więcej podsieci (moduł `subnets`).
   - `google_compute_firewall.rules` – reguły firewalla dla ruchu przychodzącego i wychodzącego (`module.firewall_rules`).
   - `google_compute_firewall.default-internal-allow-all` – domyślna reguła dopuszczająca ruch wewnątrz sieci.
   - `google_compute_firewall.fw-allow-ingress-from-iap` – pozwala na dostęp z IAP (Identity-Aware Proxy).
   - `google_compute_shared_vpc_host_project.shared_vpc_host` – ustawia projekt jako host dla Shared VPC.
   - `google_compute_router.router` + `google_compute_router_nat.nats` – umożliwiają dostęp do internetu dla zasobów bez zewnętrznych IP (moduł `cloud-router`).
   - `google_compute_route.route` – niestandardowe trasy (moduł `routes`).

   Zależności w tym module przedstawia poniższy graf:
   
   ![vpc-graph.jpg](doc/figures/vpc-graph.jpg)

7. Reach YARN UI
    
   Tunel ustawiono przy urzyciu komendy:
   ```
   bash gcloud compute ssh cluster-tbd1-m \ 
   --project=tbd-2025l-313577 \ 
   --zone=europe-west1-b \ 
   -- -L 8088:localhost:8088 
   ```

   ![yarn.jpg](doc/figures/yarn.jpg)

8. Draw an architecture diagram (e.g. in draw.io) that includes:
    1. VPC topology with service assignment to subnets
    2. Description of the components of service accounts
    3. List of buckets for disposal
    4. Description of network communication (ports, why it is necessary to specify the host for the driver) of Apache Spark running from Vertex AI Workbech
  
    ![tbd_vpc.png](doc/figures/tbd_vpc.png)

9. Create a new PR and add costs by entering the expected consumption into Infracost
For all the resources of type: `google_artifact_registry`, `google_storage_bucket`, `google_service_networking_connection`
create a sample usage profiles and add it to the Infracost task in CI/CD pipeline. Usage file [example](https://github.com/infracost/infracost/blob/master/infracost-usage-example.yml) 

    ```yaml
    version: 0.1
    resource_usage:
      google_artifact_registry_repository:
        storage_gb: 32
        monthly_egress_data_transfer_gb:
          europe_west1: 8
      google_storage_bucket:
        storage_gb: 128
        monthly_class_a_operations: 4096
        monthly_class_b_operations: 8192
        monthly_data_retrieval_gb: 64
        monthly_egress_data_transfer_gb:
          same_continent: 32
          worldwide: 32
          asia: 0
          china: 0
          australia: 0
      google_service_networking_connection:
        monthly_egress_data_transfer_gb:
          same_region: 32
          worldwide: 32
          europe: 32
          us_or_canada: 0
          asia: 0
          south_america: 0
          oceania: 0
    ```

   ![infracost.jpg](doc/figures/infracost.jpg)

10. Create a BigQuery dataset and an external table using SQL

    Utworzenie schemy

    ![create_schema_sql.png](doc/figures/create_schema_sql.png)

    ![schema_created.png](doc/figures/schema_created.png)

    Utworzenie tabeli

    ![create_table_sql.png](doc/figures/create_table_sql.png)

    ![table_created.png](doc/figures/table_created.png)

    Select z tabeli

    ![select_sql.png](doc/figures/select_sql.png)

    Wyniki

    ![select_result.png](doc/figures/select_result.png)

    Wynik można zobaczyć w różnym formacie, np. histogram

    ![select_result_chart.png](doc/figures/select_result_chart.png)

    <br/> 
    ### **ORC jest formatem kolumnowym, który zawiera schemat danych w samym pliku.** 

11. Find and correct the error in spark-job.py

   Znaleziono błąd w logach tego joba:
   
   ![spark-job-error.jpg](doc/figures/spark-job-error.jpg)
   
   Widnieje tutaj informacja, że "Specified bucket does not exist". Należało 
   poprawić nazwę bucketa w pliku `spark-job.py`, w naszym przypadku na 
   `gs://tbd-2025l-313577-data/data/shakespeare/`. Po uruchomieniu poprawionego joba:
   
   ![spark-job-result.jpg](doc/figures/spark-job-result.jpg)

12. Add support for preemptible/spot instances in a Dataproc cluster

   [Zmieniony plik](modules/dataproc/main.tf)
       
   Kod dodany do tego pliku:
   ```
   preemptible_worker_config {
     num_instances = 2
   }
   ```
