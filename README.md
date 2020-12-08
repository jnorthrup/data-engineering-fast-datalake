this is a starting point for a datalake MVP on couchdb

how hack box session:

first install pubkey, see me for details.

then

```
ssh   -AXYC ubuntu@hackbox.somewhere.like.aws.com -L  0.0.0.0:5984:127.0.0.1:5984
```

ubuntu 18.04 is a package point of reference.

make this dir or git clone it.  

```
cd
mkdir hack
```



* decide credentials

```
export COUCHDB_USER=admin COUCHDB_SECRET=gigo CHOST=0.0.0.0
```

* get some source data


```

wget https://github.com/jpwhite3/northwind-SQLite3/blob/master/Northwind_large.sqlite.zip?raw=true
unzip Northwind_large.sqlite.zip
```


* docker 
 
install from online docs

* couchdb  

quick and insecure backdated version

```
docker stop northwind;
docker container rm northwind 
sudo rm -fr    couchdata-efs1 
docker run --name northwind -v $PWD/couchdata-efs1:/opt/couchdb/data -p 5984:5984  -d couchdb:2.3.1 
```


verify 

```
curl $CHOST:5984 
```

`{"couchdb":"Welcome","version":"2.3.1","git_sha":"c298091a4","uuid":"7f2ec45ddcc62ce8b6cf6ea9127ec2a6","features":["pluggable-storage-engines","scheduler"],"vendor":{"name":"The Apache Software Foundation"}}`


* photon 


```
uname=$COUCHDB_USER; upwd=$COUCHDB_SECRET; \
couch="-H Content-Type:application/json -X PUT http://$uname:$upwd@$CHOST:5984/photon"; \
curl $couch; curl https://raw.githubusercontent.com/ermouth/couch-photon/master/photon.json | \
curl $couch/_design/photon -d @- ; curl $couch/_security -d '{}' ; \
couch=''; uname=''; upwd=''
```
for couch pre-3.x


```
uname=$COUCHDB_USER; upwd=$COUCHDB_SECRET; \
couch="-H Content-Type:application/json -X PUT http://$CHOST:5984/photon"; \
curl $couch; curl https://raw.githubusercontent.com/ermouth/couch-photon/master/photon.json | \
curl $couch/_design/photon -d @- ; curl $couch/_security -d '{}' ; \
couch=''; uname=''; upwd=''
```
 

play around:

```
http://127.0.0.1:5984/photon/_design/photon/index.html
```

* jdbc2json

install zulu openjdk

here:
https://docs.azul.com/zulu/zuludocs/ZuluUserGuide/PrepareZuluPlatform/AttachAzulPackageRepositories.htm

```
sudo apt install zulu15-jdk-headless

cd
git clone https://github.com/jnorthrup/jdbc2json.git
pushd jdbc2json
./mvnw install
popd
cd hack

TERSE=true ../jdbc2json/bin/jdbctocouchdbbulk.sh http://$COUCHDB_USER:$COUCHDB_SECRET@$CHOST:5984/northwind_ jdbc:sqlite:Northwind_large.sqlite




```
validate: pull a CSV from json view

```
(
 curl -s 0.0.0.0:5984//northwind_employeeterritory/_design/meta/_view/asMap?include_docs=true >tmp1 

jq<tmp1 '.rows[0].value | keys_unsorted  | [@csv] '
jq<tmp1 '.rows[].doc?.row? |[@csv]' 
) | tr -d '\\"'
  | tr --squeeze-repeats '\n'

```


