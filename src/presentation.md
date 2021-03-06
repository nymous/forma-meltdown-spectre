name: inverse
layout: true
class: center, middle, inverse

---

# {{{title}}}
### {{{subtitle}}}

.attacks-img[
  ![Meltdown logo](./assets/img/meltdown_logo.svg)
  ![Spectre logo](./assets/img/spectre_logo.svg)
]

#### {{{ link }}}

---
layout: false

.left-column[
## Les faits
]

.right-column[
* 3 janvier 2018 : fuite des infos sur des nouvelles failles probables, après des découvertes dans le code du kernel Linux

.redacted-img[
  ![Redacted Linux comment](./assets/img/redacted_linux.png)
]
]

---
count: false

.left-column[
## Les faits
]

.right-column[
* 3 janvier 2018 : fuite des infos sur des nouvelles failles probables, après des découvertes dans le code du kernel Linux
]

.right-column-no-padding[
* 3 janvier 2018 toujours : divulgation au public de 2 failles découvertes sur les CPU, Meltdown et Spectre ([meltdownattack.com](meltdownattack.com) et [spectreattack.com](spectreattack.com))
]

--
count: false

.right-column-no-padding[
* Remontons le temps... juin 2017 : découverte de la faille indépendamment par Google, Cyberus Technology, et l'université de technologie de Graz

* Intel, AMD et ARM sont informés

* Publication coordonnée prévue le 9 janvier 2018... Oups ^^
]

---
template: inverse

# Un CPU ça va vachement vite...
## Comment ?

---

.left-column[
## Un CPU moderne
]

.right-column[
.img[
![CPU architecture](./assets/img/cpu_architecture.png)
]
]

---
count: false

.left-column[
## Un CPU moderne
]

.right-column[
* Processeur scalaire

```python
t = a+b
u = c+d
v = e+f
w = v+g
x = h+i
y = j+k
```

6 instructions ↔ 6 cycles CPU
]

---
count: false

.left-column[
## Un CPU moderne
]

.right-column[
* Processeur scalaire
]

.right-column-no-padding[
* Processeur super-scalaire

.code-multi-columns[
```python
t = a+b
u = c+d
v = e+f
w = v+g
x = h+i
y = j+k
```
➡
```python
t, u = a+b, c+d
v    = e+f         # second pipe does nothing here
w, x = v+g, h+i
y    = j+k
```
]
6 instructions ↔ 4 cycles CPU
]

---
count: false

.left-column[
## Un CPU moderne
]

.right-column[
* Processeur scalaire
* Processeur super-scalaire
]

.right-column-no-padding[
* Out-of-order execution

.code-multi-columns[
```python
t = a+b
u = c+d
v = e+f
x = h+i
w = v+g
y = j+k
```
➡
```python
t, u = a+b, c+d
v, x = e+f, h+i
w, y = v+g, j+k
```
]

6 instructions ↔ 3 cycles CPU
]

---
count: false

.left-column[
## Un CPU moderne
]

.right-column[
* Processeur scalaire
* Processeur super-scalaire
* Out-of-order execution
]

.right-column-no-padding[
* Prédiction de branche et exécution spéculative

.code-multi-columns[
```python
t = a+b
u = t+c
v = u+d
if v:
   w = e+f
   x = w+g
   y = x+h
```
➡
```python
t = a+b
u = t+c
v = u+d
w_ = e+f
x_ = w_+g
y_ = x_+h
if v:
    w, x, y = w_, x_, y_
```
➡
```python
t, w_ = a+b, e+f
u, x_ = t+c, w_+g
v, y_ = u+d, x_+h
if v:
    w, x, y = w_, x_, y_
```
]

4 ou 7 cycles ➡ ~3 cycles
]

---
count: false

.left-column[
## Un CPU moderne
]

.right-column[
* Processeur scalaire
* Processeur super-scalaire
* Out-of-order execution
* Prédiction de branche et exécution spéculative
]

.right-column-no-padding[
* Cache

Une instruction en 0.5ns (Raspberry Pi 3)

➡ La RAM c'est leeeeeeeeent (20-100ns)

➡ Cache sur le CPU des données récentes
]

---
template: inverse

# Exploitation

---

.left-column[
## Exploitation
]

.right-column[

```python
t = a+b
u = t+c
v = u+d
if v:
*   w = kern_mem[address]   # if we get here, fault
    x = w&0x100
    y = user_mem[x]
```
⬇ (si la prédiction de branche est habituée à `v` vrai)
```python
t, w_ = a+b, kern_mem[address]
u, x_ = t+c, w_&0x100
*v, y_ = u+d, user_mem[x_]

if v:
    # fault
    w, x, y = w_, x_, y_      # we never get here
```

Si `v` est faux (`v=0`), `user_mem[x_]` aura chargé soit `0x000` soit `0x100`

➡ On mesure le temps d'accès à toutes les valeurs de `user_mem[]`, la plus rapide donne le contenu chargé depuis le kernel
]

---
template: inverse

# Qui est touché ?

---

.left-column[
## Victimes
]

---
count: false

.left-column[
## Victimes
]

.right-column[
  * Meltdown : Intel (+ un CPU ARM)
]

--
count: false

.right-column-no-padding[
  * Spectre: Intel, AMD, ARM
]

--
count: false

.right-column-no-padding[
La conception même des CPU depuis 1995 est à changer

**Tu es touché, tu es touché, tout le monde est touché !**

.img[
![You get a car gif](https://media1.tenor.com/images/fffafc10d00f6adc481989adbc0ee4f0/tenor.gif)
]
]

---
template: inverse

# Comment se protéger ?

---

.left-column[
## Le patch
]

.right-column[
.img[
![Kernel address space après patch](./assets/img/kernel_apres_patch.png)
]

Séparer les espaces mémoires kernel et applications

**Problème :** beaucoup de changement d'adressage, vidage et remise en cache ➡ perte de performances selon les applications
]

---
count: false

.left-column[
## Le patch
## Se protéger
]

.right-column[
  * Linux : kernel 4.14.11, 4.15-rc6, backporté vers 4.9.75 LTS et 4.4.110 LTS

  * Distributions : Debian OK, Ubuntu OK, RedHat OK, CentOS OK, ArchLinux OK

  * Virtualisation : VMware **Pas OK**, XEN **Pas OK**, QEMU **Pas officiel**, KVM **Bientôt**
]

--
count: false

.right-column-no-padding[
  * Apple : patché en douce dans iOS 11.2, macOS 10.13.2, et tvOS 11.2
]

--
count: false

.right-column-no-padding[
  * Windows : mise à jour dispo, mais ⚠️ dépend de votre antivirus !! S'il n'est pas à jour, vous ne recevres plus **aucune** mise à jour !
]

--
count: false

.right-column-no-padding[
  * Navigateurs : Firefox OK, Chrome **pas OK** (le 23 janvier), Internet Explorer et Edge OK
]

--
count: false

.right-column-no-padding[
  * Cloud : Amazon AWS rebooté OK, Google Cloud rebooté OK, Microsoft Azure rebooté OK
]

---
template: inverse

# On m'avait promis une blague !

---

.blague-img[
  ![Blague CPU](./assets/img/blague-tweet.png)
]

---
template: inverse

# The end

### {{{ link }}}

---

.left-column[
## Sources
]

.right-column[
* Wikipédia : [Meltdown][Wikipedia Meltdown], [Spectre][Wikipedia Spectre]

* [Comparaison avec un fast food](https://dev.to/isaacandsuch/how-meltdown-works-28j2)

* [Article de l'ANSSI](https://www.cert.ssi.gouv.fr/actualite/CERTFR-2018-ACT-001/)

* [Article de Raspberry-Pi expliquant la faille](https://www.raspberrypi.org/blog/why-raspberry-pi-isnt-vulnerable-to-spectre-or-meltdown/) (simple à comprendre)

* [Récapitulatif des patchs disponibles](https://github.com/hannob/meltdownspectre-patches/)

* Proof-of-Concept : https://github.com/IAIK/meltdown/ — https://github.com/speed47/spectre-meltdown-checker — https://github.com/gentilkiwi/spectre_meltdown — https://github.com/paboldin/meltdown-exploit — https://github.com/Eugnis/spectre-attack

* Articles en anglais [The Register](https://www.theregister.co.uk/2018/01/02/intel_cpu_design_flaw/) — [Sophos - Naked Security](https://nakedsecurity.sophos.com/2018/01/03/fckwit-aka-kaiser-aka-kpti-intel-cpu-flaw-needs-low-level-os-patches/) — [Razorpay](https://razorpay.com/blog/meltdown-paper-summary/) — [Google Project Zero 1](https://security.googleblog.com/2018/01/todays-cpu-vulnerability-what-you-need.html) — [Google Project Zero 2](https://googleprojectzero.blogspot.fr/2018/01/reading-privileged-memory-with-side.html) et en français : [Hackndo - Pixis](http://beta.hackndo.com/meltdown-spectre/) (article excellent) — [Xhark](http://blogmotion.fr/internet/securite/metldown-spectre-comprendre-16865) — [LinuxFR](https://linuxfr.org/news/deux-failles-critiques-meltdown-et-spectre)

* [Mailing-list Linux Kernel](https://lkml.org/lkml/2017/12/4/709)
]

[Wikipedia Meltdown]: https://en.wikipedia.org/wiki/Meltdown_(security_vulnerability)
[Wikipedia Spectre]: https://en.wikipedia.org/wiki/Spectre_(security_vulnerability)
