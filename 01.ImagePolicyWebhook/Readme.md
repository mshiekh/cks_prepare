# ImagePolicyWebhook

![1](../images/1.png)

## official documentation：https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook

## <font color=red>master node configurations</font>
## 0、epconfig directory
```shell
admission_config.yaml  apiserver-client-cert.pem  apiserver-client-key.pem  external-cert.pem  external-key.pem  kubeconf
```

## 1、update apiserver.yaml
```yaml
- --enable-admission-plugins=NodeRestriction,ImagePolicyWebhook
- --admission-control-config-file=/etc/kubernetes/epconfig/admission_config.yaml
```
## 2、create volume and volume mount
```yaml
# volume mount
    volumeMounts:
    - mountPath: /etc/kubernetes/epconfig
      name: epconfig
# volumes      
  volumes:
    - name: epconfig
    	hostPath:
      	path: /etc/kubernetes/epconfig
```

## 3、edit the admission_config.yaml 
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: ImagePolicyWebhook
  configuration:
    imagePolicy:
    	kubeConfigFile: /etc/kubernetes/epconfig/kubeconf #update the directory
      allowTTL: 50
      denyTTL: 50
      retryBackoff: 500
      defaultAllow: flase #update to false
```

## 4 update kubeconf file
```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/epconfig/external-cert.pem  # CA for verifying the remote service.
    server: https://acme.local:8082/image_policy # URL of remote service to query. Must use 'https'.
  name: image-checker

contexts:
- context:
    cluster: image-checker
    user: api-server
  name: image-checker
current-context: image-checker
preferences: {
    }

# users refers to the API server's webhook configuration.
users:
- name: api-server
  user:
    client-certificate: /etc/kubernetes/epconfig/apiserver-client-cert.pem # cert for the webhook admission controller to use
    client-key:  /etc/kubernetes/epconfig/apiserver-client-key.pem # key matching the cert
```

## 5、restart kubelet
```shell
systemctl daemon-reload

systemctl restart kubelet
```
