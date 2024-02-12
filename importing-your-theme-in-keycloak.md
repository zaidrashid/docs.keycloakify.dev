# 📥 Importing your theme in Keycloak

Now that you have your theme as a .jar file, let's see how you can import it in Keycloak so that it apprer  in the dropdown list for selecting theme in the Keycloak Admin console. &#x20;

<figure><img src=".gitbook/assets/image (16).png" alt="" width="375"><figcaption><p>Custom login and acount theme selecte in the Keycloak Admin console</p></figcaption></figure>



{% hint style="info" %}
Attention: Many pepole miss this but **Keycloakify provides a way to test your theme in a local container just by running a single command**.  \
You just have to follow the instruction printed in the console after running running the `build-keycloak-theme` command!  \
[Video tutorial](https://www.youtube.com/watch?v=WMyGZNHQkjU).
{% endhint %}

{% tabs %}
{% tab title="Using CloudIAM" %}
If you are utilizing a Keycloak instance managed by [CloudIAM](https://cloud-iam.com/?mtm\_campaign=keycloakify-deal\&mtm\_source=keycloakify-doc-header), importing themes and extensions is quite straightforward.

{% hint style="info" %}
Uploading custom JAR files is only available with paid plans.&#x20;

If you decide to subscribe, please consider using the code `keycloakify5`.&#x20;

This code will provide you with a 5% discount, and we will also receive 5%, which greatly supports our project!
{% endhint %}

{% embed url="https://app.tango.us/app/embed/fe0030b4-598b-4c3a-8d9e-a6710d337749" %}
{% endtab %}

{% tab title="Using Docker" %}
Let's see how you would go about creating a Keycloak Docker image with your theme available.\


{% embed url="https://willwill96.github.io/the-ui-dawg-static-site/en/keycloakify/#integrating-keycloak-and-keycloakify-jar" %}
Checkout this great tutorial that explains it in great details
{% endembed %}

```bash
cd ~/github
mkdir docker-keycloak-with-theme
cd docker-keycloak-with-theme
git clone https://github.com/keycloakify/keycloakify-starter
cd keycloakify-starter
# Just to make sure theses instruction remain relevent in the future
# We pin the version of the starter we are using.  
git checkout d28c986686c3255b812152f68f3458995fa8bfb7
cd ..

cat << EOF > ./Dockerfile
FROM node:18 as keycloakify_jar_builder
RUN apt-get update && \
    apt-get install -y openjdk-17-jdk && \
    apt-get install -y maven;
COPY ./keycloakify-starter/package.json ./keycloakify-starter/yarn.lock /opt/app/
WORKDIR /opt/app
RUN yarn install --frozen-lockfile
COPY ./keycloakify-starter/ /opt/app/
RUN yarn build-keycloak-theme

FROM quay.io/keycloak/keycloak:latest as builder
WORKDIR /opt/keycloak
COPY --from=keycloakify_jar_builder /opt/app/build_keycloak/target/keycloakify-starter-keycloak-theme-5.1.3.jar /opt/keycloak/providers/
RUN /opt/keycloak/bin/kc.sh build

FROM quay.io/keycloak/keycloak:latest
COPY --from=builder /opt/keycloak/ /opt/keycloak/
ENV KC_HOSTNAME=localhost
ENTRYPOINT ["/opt/keycloak/bin/kc.sh", "start-dev"]
EOF

docker build -t docker-keycloak-with-theme .
docker run \
    -e KEYCLOAK_ADMIN=admin \
    -e KEYCLOAK_ADMIN_PASSWORD=admin \
    -p 8080:8080 \
    docker-keycloak-with-theme
```
{% endtab %}

{% tab title="Using Helm" %}
If you use [Bitnami's Keycloak Helm chart](https://github.com/bitnami/charts/tree/main/bitnami/keycloak) you can levrage the initContainers parameter to load your theme. &#x20;

{% code title="keycloak-values.yaml" %}
```yaml
...  
initContainers: |
  - name: realm-ext-provider
    image: curlimages/curl
    imagePullPolicy: IfNotPresent
    command:
      - sh
    args:
      - -c
      - |
        curl -L -f -S -o /extensions/keycloak-theme.jar https://github.com/keycloakify/keycloakify-starter/releases/download/latest/keycloak-theme.jar
    volumeMounts:
      - name: extensions
        mountPath: /extensions

extraVolumeMounts: |
  - name: extensions
    mountPath: /opt/bitnami/keycloak/providers

extraVolumes: |
  - name: extensions
    emptyDir: {}
```
{% endcode %}

Of course you need to replace `github.com/keycloakify/keycloakify-starter/releases/download/latest/keycloak-theme.jar` by an URL of your theme.  \
\
Read [this section of the starter project readme](https://github.com/keycloakify/keycloakify-starter?tab=readme-ov-file#the-ci-workflow) to learn how to get GitHub Action to publish your theme's JAR as assets of your GitHub release.
{% endtab %}

{% tab title="Bare metal" %}
What you need to know is that your keycloak-theme.jar should be placed in the provider directory of your Keycloak (e.g: `/opt/keycloak/providers)`\
After that you should run bin/kc.sh build (e.g: `bash /opt/keycloak/bin/kc.sh build`)

Then you can start your Keycloak server, your theme should be available in it!&#x20;
{% endtab %}
{% endtabs %}
