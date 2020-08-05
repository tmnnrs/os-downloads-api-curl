# OS Downloads API (shell script + cURL)

Shell script which can be used to request Ordnance Survey datasets from the [OS Downloads API](https://osdatahub.os.uk/docs/downloads/overview) (available through the OS Data Hub).

This is primarily for \*NIX users – but is an alternative to the *Node.js* script in the [OS Downloads API: Getting started guide](https://osdatahub.os.uk/docs/downloads/gettingStarted).

### Inputs

name | type | description
--- | --- | ---
`productId` | *string* | The id of the product.
`area` | *array* | Filter the list of downloads to only include those which cover this area. 'GB' is all of Great Britain, while codes like 'HP' cover National Grid Reference squares.
`format` | *string* | Filter the list of downloads to only include those with this format.
`subformat` | *string* | [Optional] Filter the list of downloads to only include those with this subformat.

NOTE: The `format` and `subformat` values should be URL encoded (which converts characters into a format that can be transmitted over the Internet).

Further information about the methods used to call the OS Downloads API can be in the [OS Downloads API: Technical specification](https://osdatahub.os.uk/docs/downloads/technicalSpecification).

Alternatively – **GET** `/products` can be used to return a list of the products that are available to download:
https://api.os.uk/downloads/v1/products?expanded=true
 
### Process

The basic premise is as follows:
- Loop through each of the values in the `area` array; and for each value use cURL to fetch the headers from the OS Downloads API **GET** `/products/{productId}/downloads` (via the 'redirect' parameter).
- Pipe the response (see example below) into GREP in order to extract the `Location:` string.
```
HTTP/1.1 307 Temporary Redirect
Date: Sat, 25 Jul 2020 20:41:22 GMT
Content-Type: application/json;charset=UTF-8
Content-Length: 0
Connection: keep-alive
Location: https://omseprd1stdstordownload.blob.core.windows.net/downloads/OpenTOID/2020-06/100kmSquares/CSV/osopentoid_202006_csv_hp.zip?sv=2018-11-09&spr=https&se=2020-07-25T21%3A41%3A22Z&sr=b&sp=r&sig=4qKPgQaYI393kqMzuZDF0YzdqeFhSjHewiS9A1u0EH0%3D
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: origin, x-requested-with, accept
Access-Control-Max-Age: 3628800
Access-Control-Allow-Methods: GET
```
- Use AWK to split the Location: string (using the '?') and store the URL (variable A) and request parameters (variable B).
- Echo the request parameters; and pipe them into a cURL **GET** command to output/download the relevant ZIP file.

### script.sh

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
