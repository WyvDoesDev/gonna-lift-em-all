# HTB Hack The Boo gonna-lift-em-all 


### The Code 

```py
from Crypto.Util.number import bytes_to_long, getPrime
import random

FLAG = b'HTB{??????????????????????????????????????????????????????????????????????}'

def gen_params():
  p = getPrime(1024)
  g = random.randint(2, p-2)
  x = random.randint(2, p-2)
  h = pow(g, x, p)
  return (p, g, h), x

def encrypt(pubkey):
  p, g, h = pubkey
  m = bytes_to_long(FLAG)
  y = random.randint(2, p-2)
  s = pow(h, y, p)
  return (g * y % p, m * s % p)

def main():
  pubkey, privkey = gen_params()
  c1, c2 = encrypt(pubkey)

  with open('data.txt', 'w') as f:
    f.write(f'p = {pubkey[0]}\ng = {pubkey[1]}\nh = {pubkey[2]}\n(c1, c2) = ({c1}, {c2})\n')


if __name__ == "__main__":
  main()
  ```
## Supplied values

```
p = 163096280281091423983210248406915712517889481034858950909290409636473708049935881617682030048346215988640991054059665720267702269812372029514413149200077540372286640767440712609200928109053348791072129620291461211782445376287196340880230151621619967077864403170491990385250500736122995129377670743204192511487
g = 90013867415033815546788865683138787340981114779795027049849106735163065530238112558925433950669257882773719245540328122774485318132233380232659378189294454934415433502907419484904868579770055146403383222584313613545633012035801235443658074554570316320175379613006002500159040573384221472749392328180810282909
h = 36126929766421201592898598390796462047092189488294899467611358820068759559145016809953567417997852926385712060056759236355651329519671229503584054092862591820977252929713375230785797177168714290835111838057125364932429350418633983021165325131930984126892231131770259051468531005183584452954169653119524751729
(c1, c2) = (159888401067473505158228981260048538206997685715926404215585294103028971525122709370069002987651820789915955483297339998284909198539884370216675928669717336010990834572641551913464452325312178797916891874885912285079465823124506696494765212303264868663818171793272450116611177713890102083844049242593904824396, 119922107693874734193003422004373653093552019951764644568950336416836757753914623024010126542723403161511430245803749782677240741425557896253881748212849840746908130439957915793292025688133503007044034712413879714604088691748282035315237472061427142978538459398404960344186573668737856258157623070654311038584)
```

From looking at the code we can infer this is [Diffie-Hellman 1024](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)
Also, from the supplied values that we have we can solve for other variables, our end goal would be m since that's our flag in long format.

lets solve for the other variables:
y should be very easy to solve for, since y = c1 * g^-1 mod p
s can be easily solved by computing s = h^y mod p
Finally m = c2 * s^-1 mod p

### Solving for y
To solve for y we need to first get the modular multiplicate inverse of g^-1 for this I used this [tool](https://www.dcode.fr/modular-inverse) with the int being g and the modulo being p, if you did this right you should get 
```
g^-1 = 120027004247158358184703385511138910446176598283657810928960020555251889532032199706156913358525135228299658796007082082987316875751452608872617761586138905964991747541264336966530405406630206297358091931611374901221899003603216345652222991753618659380928999922962044386202238694636990131574221328099007640482
```

To make sure it's correct, you can do g^=1 * g mod p if this equation equals 1 then you have the right inverse

Once you have that number you want to continue solving for y so y = c1 * g^-1 mod p

```
y = 151545036818752418931716093171030939827729309717327611184964755063685533596024474465903219353892430936128129116061427826165388249908655823309049171719865481058072839169911183783187254412879190149192386989186799988830028288993778261809217410313001568877314905167838867719115514855795015291428405597461040625720
```

## Solving for s

s is really easy to solve for since it's h^y mod p and we know all those values. I recommend using pythons pow() function to make the math on this one not take forever.
```
s = 97462626764574972789405707853736776801131892662685049788888445937335307309802916804770978800211152464507610133907443690200443337122554845143013035673411159832257337734583042568923321169807909583339712803034130755892624097871888129173372595909172265258031320357247928751965375753164262717332601963215413213638
```

## Solving for m 

Solving for m should be very similar to what we did solving for y, we are even using the same tool as before, but this time we want our int being used and s and keeping the modulo as p, for s^-1 you should get
 
```
s^-1 = 31346328967915532437069190021834034870416234511102747411963727347528808773951209443402276830477716072945708394322703296371784510296678805269713052825970582951827590069078060945380207673640183443015571912058129616320846896831349884362934154099966370303518207509550815372317213136611174672595234824291719313740
``` 


from here m = c2 * s^-1 mod p which should be the flag in a long
```
flag = 1172386289712688621866206342757024282557431573799768202628558217825308016488998421960879829861191968014842977524818155697111668467803322833848788605649390583219898324267188549415037
```

## Solving for the flag

Now we technically have the flag, but in order to get it into the flag format we need to use the opposite of bytes_to_long which is long_to_bytes
we will use the same crypto library as the code

```py
from Crypto.Util.number import long_to_bytes

flag = b'1172386289712688621866206342757024282557431573799768202628558217825308016488998421960879829861191968014842977524818155697111668467803322833848788605649390583219898324267188549415037'

print(long_to_bytes(flag))

```

that should return the flag 
```
HTB{b3_c4r3ful_wh3n_1mpl3m3n71n6_cryp705y573m5_1n_7h3_mul71pl1c471v3_6r0up}
```

