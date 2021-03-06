# motif-broker-request

Example in test/test.py

## Configure motif-broker-request

```python
import motif_broker_request.request as mb_request

mb_request.configure("http://localhost:5984") #Default is localhost:3282
```

You can also change bulk length 
```python
mb_request.set_bulk_length(10000) #Default is 50000
```

## Request motif-broker 

```python
sgrnas = ["AAAAAAAAAAAAAAAAAAATGGG", "TCCAAAAAAAAACAGTGGATTGG", "CACTAAAAAAGAAGACCAAGCGG"] # sgRNAs you want to search
res = mb_request.get(sgrnas)

print(res)
```
```
>>> {
    'AAAAAAAAAAAAAAAAAAATGGG': {
        'dd6cfb980c8a3659acffa4f002ea7404': {
            'NZ_LR214986.1': ['+(575298,575320)']
        }
    },
    'TCCAAAAAAAAACAGTGGATTGG': {
        'dd6cfb980c8a3659acffa4f002f7e935': {
            'NZ_CP047242.1': ['+(3037771,3037793)']
        }
    },
    'CACTAAAAAAGAAGACCAAGCGG': {
        'dd6cfb980c8a3659acffa4f0029ff84a': {
            'NC_009785.1': ['-(1746678,1746700)']
        },
        'dd6cfb980c8a3659acffa4f002a00a06': {
            'NZ_LR134336.1': ['+(192321,192343)']
        }
    }
}
```

## Request motif-broker with some filter
You can write filter functions. This functions have to take motif_broker results as arguments and return the filtered version with same format 

**Example of function that just keep given genomes in results :**
```python
def filter_genomes(mb_res, **kwargs):
    if not "genomes" in kwargs:
        raise Exception("you must provide 'genomes' argument to get function for filter_genomes function")
    genomes = kwargs["genomes"]

    filtered_results = {}
    for sgrna in mb_res: 
        added = False 
        for org in mb_res[sgrna]:
            if org in genomes:
                if added:
                    filtered_results[sgrna][org] = mb_res[sgrna][org]
                else:
                    filtered_results[sgrna] = {org : mb_res[sgrna][org]}
    
    return filtered_results
```

Applied this way to get results : 
```python
res = mb_request.get(sgrnas, filter_predicate=filter_genomes, genomes=["dd6cfb980c8a3659acffa4f002ea7404", "dd6cfb980c8a3659acffa4f0029ff84a"])
print(res)
```
```
>>> 
{
    'AAAAAAAAAAAAAAAAAAATGGG': {
        'dd6cfb980c8a3659acffa4f002ea7404': {
            'NZ_LR214986.1': ['+(575298,575320)']
        }
    },
    'CACTAAAAAAGAAGACCAAGCGG': {
        'dd6cfb980c8a3659acffa4f0029ff84a': {
            'NC_009785.1': ['-(1746678,1746700)']
        }
    }
}
```

