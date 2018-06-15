# Setup CAS server as idP with a dev environment

The repo intends to make life easier for setting up CAS authentication server primarily as a idP along with an 
integrated dev environment to try out more customisations and integrations

The compose declares two containers

1. tomcat - servlet container to host cas (and cas-management) wars
2. maven - a dev environment with java, clone the relevant [cas-overlay](https://github.com/apereo/cas-overlay-template)

The volumes are laid out such that `tomcat/webapps` and `/etc/cas/config` are mapped to the host and both the containers

```
...
      - ${ROOT}/tomcat/webapps:/tomcat/webapps
      - ${ROOT}/tomcat/conf:/tomcat/conf
      - ${ROOT}/tomcat/certs:/tomcat/certs
      - ${ROOT}/cas/etc/cas:/etc/cas
...
      - ${ROOT}/cas/git:/cas/git
```
## Installation

1. Clone the repo on any host
`$> git clone https://github.com/meetpraveen/cas-docker-compose`
2. Set the root directory environment
```bash
$> cd cas-docker-compose
$> export ROOT="cas-docker-compose"
```
3. Clone required cas overlay repos
```bash
$> mkdir cas/git & cd cas/git
$> git clone https://github.com/apereo/cas-overlay-template.git
```
4. Apply certs for tomcat by editing the SSL Connector in tomcat/conf/server.xml
5. Start the containers
`$> docker-compose up -d`
6. Review the properties files in cas/etc/cas/config directory
7. Get inside maven container to build cas.war and deploy
```bash
$> docker exec -it maven bash
$> cd /cas/git/cas-overlay-template
$> ./build.sh package
$> cp target/cas.war /tomcat/webapps
```
8. Now you can create the service registry using json file in the format described below
`$> vim /opt/cas/etc/cas/services/idbroker-40000003.json`
```json
{
  "@class" : "org.apereo.cas.support.saml.services.SamlRegisteredService",
  "serviceId" : "^https://idbroker.webex.com/8843408e-c20f-45ea-8a23-55c1eedee439.*",
  "name" : "IdBroker",
  "id" : 40000003,
  "evaluationOrder" : 9,
  "metadataLocation" : "https://sso.idp.carehybrid.com:8443/idb-meta-praveen2-ccone.xml"
}
```
