title: Certificates

# **Certificates**

## **Kubernetes**

[source](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/04-certificate-authority.md)

Using CloudFlare's PKI toolkit, [cfssl](https://github.com/cloudflare/cfssl)

??? info "Generate Certificate Authority that can be used to generate additional TLS certificates"

    ```json title="ca-config.json"
    {
      "signing": {
        "default": {
          "expiry": "8760h"
        },
        "profiles": {
          "kubernetes": {
            "usages": ["signing", "key encipherment", "server auth", "client auth"],
            "expiry": "8760h"
          }
        }
      }
    }
    ```

    ```json title="ca-csr.json"
    {
      "CN": "Kubernetes",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Portland",
          "O": "Kubernetes",
          "OU": "CA",
          "ST": "Oregon"
        }
      ]
    }
    ```

    ```bash
    cfssl gencert -initca ca-csr.json | cfssljson -bare ca
    ```

??? info "Generate client and server certificates for each Kubernetes component and a client certificate for the Kubernetes admin user"

    Admin:

    ```json title="admin-csr.json"
    {
      "CN": "admin",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Portland",
          "O": "system:masters",
          "OU": "Kubernetes The Hard Way",
          "ST": "Oregon"
        }
      ]
    }
    ```

    ```bash
    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -profile=kubernetes \
      admin-csr.json | cfssljson -bare admin
    ```

    Client / Node certificates:

    ```bash
    for instance in worker-0 worker-1 worker-2; do
       cat > ${instance}-csr.json <<EOF
          {
            "CN": "system:node:${instance}",
            "key": {
              "algo": "rsa",
              "size": 2048
            },
            "names": [
              {
                "C": "US",
                "L": "Portland",
                "O": "system:nodes",
                "OU": "Kubernetes The Hard Way",
                "ST": "Oregon"
              }
            ]
          }
       EOF

       EXTERNAL_IP=$(gcloud compute instances describe ${instance} \
         --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')

       INTERNAL_IP=$(gcloud compute instances describe ${instance} \
         --format 'value(networkInterfaces[0].networkIP)')

       cfssl gencert \
         -ca=ca.pem \
         -ca-key=ca-key.pem \
         -config=ca-config.json \
         -hostname=${instance},${EXTERNAL_IP},${INTERNAL_IP} \
         -profile=kubernetes \
         ${instance}-csr.json | cfssljson -bare ${instance}
    done
    ```
