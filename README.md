# os-downloads-api-curl

Shell script which can be used to request Ordnance Survey datasets from the [OS Downloads API](https://osdatahub.os.uk/docs/downloads/overview) (available through the OS Data Hub).

```sh
#!/bin/zsh

productId=OpenTOID
area=( HP HT HU HW HX HY HZ NA NB NC ND NF NG NH NJ NK NL NM NN NO NR NS NT NU NW NX NY NZ OV SD SE SH SJ SK SM SN SO SP SR SS ST SU SV SW SX SY SZ TA TF TG TL TM TQ TR TV )
format=CSV

# productId=OpenGreenspace
# area=( GB )
# format=ESRI%C2%AE+Shapefile

# productId=Terrain50
# area=( GB )
# format=ASCII+Grid+and+GML+%28Grid%29

# productId=Terrain50
# area=( GB )
# format=ESRI%C2%AE+Shapefile
# subformat=%28Contours%29

for i in "${area[@]}"
do
  if [ -z ${subformat+x} ]; then
    url="https://api.os.uk/downloads/v1/products/${productId}/downloads?area=${i}&format=${format}&redirect"
  else
    url="https://api.os.uk/downloads/v1/products/${productId}/downloads?area=${i}&format=${format}&subformat=${subformat}&redirect"
  fi
  result=`curl -sI $url | grep -Fi Location`
  A=$(awk -F'?' '{gsub(/Location: /,""); print $1}' <<< $result)
  B=$(awk -F'?' '{print $2}' <<< $result)
  echo $B | curl -G -o `basename $A` -d "@-" $A
done
```
