---
layout: post
title: "Sardinas-Patterson algoritmus"
category: hungarian
---

Bevezetés
---------

A kódelméletben az egyik klasszikus probléma eldönteni egy adott kódról, hogy az egyértelműen felbontható-e kódszavak szorzatára.
Felbonthatatlan kóddal nyilván értelmetlen lenne bármit is kódolni, hisz a fogadó fél csak vakargatná a fejét, amikor megpróbálja dekódolni azt.
A Sardinas-Patterson algoritmus egyszerű megoldást nyújt annak eldöntésére, hogy egy adott változó-hosszúságú kód egyértelműen felbontható-e.

Az algoritmusról
----------------
Adott egy nemüres véges \\(A\\) halmaz a kódolandó ábécé, és egy véges \\(B\\) halmaz a kódábécé.
A továbbiakban az egyszerűség kedvéért tekintsük azokat az eseteket, ahol a kódábécénk a \\(B = \\{ 0, 1\\} \\) halmaz, azaz a bináris kódokat.
Ekkor a betűnkénti kódolás tekinthető egy \\( \phi : A \rightarrow B^* \\) leképezésnek. Egy kód akkor lesz felbontható, ha ez a \\(\phi\\) leképezés injektív.
Ha egy kódról elmondható az alábbi tulajdonságok közül bármelyik, akkor egyértelműen felbontható lesz:

  1. **Vesszős kód**: minden kódszó végén egy speciális karakter jelzi annak végét (csak itt szerepel)
  1. **Blokk kód**: minden kódszó azonos hosszúságú
  1. **Prefix kód**: egyik kódszó sem valódi kezdőszelete egyetlen *másik* kódszónak (prefixmentes)
  
Az algoritmus szempontjából az érdekes eset a harmadik.
Tekintsük a következő kódszavakat: \\( \mathcal{K} = \\{ 0, 1, 01 \\} \\).
Könnyű észrevenni, hogy a 01 kódszó előállhat kétféleképpen is, tehát a kód *nem* prefixmentes. De mi a helyzet ha a kódszavak halmaza kicsivel is bővebb?

\\[ \mathcal{K} = \\{ 010, 0001, 0110, 1100, 00011, 00110, 11110, 101011 \\} \\]

Ebben az esetben már ránézésre sem könnyű megállapítani a kódról, hogy prefixmentes-e és itt jön a képbe a Sardinas-Patterson algoritmus.
Az algoritmus fő mozgatórugója megállapítani, hogy van-e olyan kód, amely többféleképpen bomlik fel kódszavak szorzatára, ha találunk ilyet készen vagyunk és elmondhatjuk, hogy a kód nem felbontható.
Ehhez bevezetjük a következő jelölést: \\[ Q^{-1}P = \\{  y | xy \in P \wedge x \in Q\\} \\]
Ennek a halmaznak azok a \\(P\\)-beli "maradék" sztringek lesznek az elemei, amelyek valamely \\(Q\\)-beli prefix eltávolításával állnak elő. \\
Az algoritmus első lépése kiszámítani az önmagával vett maradékhalmazt. Jelentse most itt \\(\lambda\\) az üres szót.
\\[ U_{1} = \mathcal{K}^{-1}\mathcal{K} \setminus \lambda \\]
Minden további halmazt generáljuk az alábbi módon.
\\[ U_{i+1} = U_{i}^{-1}\mathcal{K} \cup \mathcal{K}^{-1}U_{i} \, \(\forall i \ge 1\) \\]
A tétel szerint \\( \mathcal{K} \\) kód akkor és csak akkor, ha \\( \mathcal{K} \cap U_{i} = \emptyset, \, \forall i \ge 1.\\)
Más szóval \\( \mathcal{K} \\) kód akkor és csak akkor, ha \\( \lambda \notin U_{i} \, \(\forall i \ge 1\).\\)
\\(U_{i+1}\\) definíciójából adódik, hogy \\(\lambda \in U_{i+1}\\) ha \\( \mathcal{K} \cap U_{i+1} \ne \emptyset.\\)
Ha az üres szó megjelenik a halmazunkban, az azt jelenti, hogy találtunk egy "tanút" arra az esetre, amikor egy kód
nem bomlik fel egyértelműen kódszavak szorzatára és az algoritmus hamis üzenettel tér vissza \\(\mathcal{K}\\) felbonthatóságát illetően.
Az algoritmus akkor tér vissza igazzal, ha \\( \exists j < i : \, U_{j} = U_{i}\\), mivel tudjuk, hogy a 
\\[ U_1, U_2, \dots , U_n\\] sorozat ciklikus valamely \\(n\\)-re. A bizonyításra itt most nem kerül sor, részleteiben elolvasható [1] 3.1 fejezetében.

Implementáció
-------------
Egy lehetséges implementáció Scala-ban. Az \\(U_{i+1}\\) halmazok előállítása
nagyon jól programozható rekurzív megoldással.

```scala
import scala.annotation.tailrec

object SardinasPattersenAlg {

  private val EmptyString = ""

  def isUniquelyDecodable(code: Set[String]): Boolean = {
    @tailrec
    def helper(prevs: Seq[Set[String]]): Boolean = {
      val last = prevs.last
      if (last contains EmptyString) false
      else {
        val next = leftQuotient(last, code) union leftQuotient(code, last)
        if (next.isEmpty || prevs.contains(next)) return true
        helper(prevs :+ next)
      }
    }

    val first = leftQuotient(code, code) - EmptyString
    helper(Seq(first))
  }

  def leftQuotient(first: Set[String], second: Set[String]): Set[String] =
    for (a <- second; b <- first; if b.startsWith(a)) yield {
      if (a == b) EmptyString else b.substring(a.length)
    }
}
```

A teszt nyílván nem kimerítő, inkább csak egy "proof-of-concept", amely bemutatja a ScalaTest framework lehetőségeit.

```scala
import org.scalatest.{FlatSpec, Matchers}

class ExampleSpec extends FlatSpec with Matchers {

  def code = Set[String]("010", "0001", "0110", "1100", "00011", "00110", "11110", "101011")

  "SardinasPattersenAlg" should "compute that code is not unique" in {
    SardinasPattersenAlg.isUniquelyDecodable(code) should be(false)
  }
}
```

Irodalomjegyzék
---------------

  1. [Sardinas-Patterson like algorithms in coding theory](http://www.engr.uconn.edu/~bop04002/data/licenta.pdf), Bogdan Pasaniuc, Iasi, 2013
  1. Bevezetés a matematikába, Járai Antal, Budapest, 2009
  1. [Wikipedia szócikk](https://en.wikipedia.org/wiki/Sardinas%E2%80%93Patterson_algorithm)


