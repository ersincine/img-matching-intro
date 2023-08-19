# Benim Önsözüm
- **About these notes**: Emre icin, Ersin tarafindan. Kitap cok kapsamli. Bize gerekli yerleri tespit ettim ve daha anlasilir bir sekilde yazmaya calistim. Ayni basliklara kitaptan kendin de calisabilirsin ama bayagi zor olur. Anlamakta zorlandigin spesifik yerler olursa onlara oradan bakmak daha iyi bir fikir. Ozet ya da direkt tercume yapmadim, bol bol kendi yorumlarimi, aciklamalarimi ekledim.
- **About "multiple view geometry"**: Normalde geometri tek bir bakis acisiyla ilgilidir. Mesela lisede bir geometri sorusuyla karsilasinca soru tek bir bakis acisindan verilir. Ancak biz birden fazla bakis acisiyla ayni sahneye (veya nesneye) bakiyoruz. Bunun computer vision icindeki karsiligi aslinda ayni sahnenin (veya nesnenin) birden fazla bakis acisindan cekilmis fotograflariyla ugrasmak.
	- Birden fazla bakis acisi sayesinde yapilabilen bircok uygulama var. Ornegin (kolaydan zora siralarsak):
		- Image stitching: Birden fazla fotografi birlestirmek, mesela panoramik fotograf olusturmak. (Ornegin http://matthewalunbrown.com/autostitch/autostitch.html)
		- Structure-from-Motion (SfM): Bir insanin, nesnenin, binanin, sehrin vb. 3 boyutlu modelini cikarmak. (Rastgele turist fotograflarindan bile olabilir. Illa ozel bir kamerayla gidip her acidan fotograf cekmek falan sart degil.)
		- Simultaneous Localization And Mapping (SLAM): Haritayi bilmeyen bir robotun (otonom arac da olabilir, drone falan da olabilir) gezerken hem haritayi cikarmasi hem de haritanin neresinde oldugunu takip edebilmesi. (Hem ardisik frameler uzerinden ne tarafa dogru ve ne kadar gittigimiz takip edilebilir hem de gecmis framelerle karsilastirma yaparak donup dolasip ayni yere gelip gelmedigimiz takip edilebilir)
	- Bunlarin hepsi de birden fazla imgeyi eslestirebilmeyi (image matching) gerektiriyor. (Yani bunlar applications of image matching.)
	- Bu kitap hem image matching'in farkli hallerini hem de image matching sonrasinda gereken islemleri tariyor.
- **About our project**: 
	- Sadece image matching ile ilgileniyoruz (spesifik bir uygulama ile degil). 
	- Sadece 2 imgeli durumu calisiyoruz. Elimizde 2'den fazla imge varsa bile bunlari 2'ser 2'ser eslestirebiliriz. (Tabii butun pairleri tek tek eslestirmek verimlilik anlaminda ideal degil.)
		- Buna stereo image matching de deniyor. Ama image matching denince normalde anlasilan da bu. Ikiden fazla imgeyi beraber eslestirmek multi-view image matching diye geciyor.
	- Fotograflardaki nesnelerimiz duzlemsel (hatta genelde tek bir duzlemsel nesne). Girdimiz iki tane imge. Ciktimiz bir homography matrix.
		- Bazen nesnelerimiz 3 boyutlu da olabiliyor ama uzaktan cekilen fotograflar oluyor. Uzaktan bakinca 2 boyutlu gibi gozukuyorlar (derinlikleri mesafenin yaninda yok gibi kaliyor), o yuzden duzlemlermis gibi dusunebiliyoruz.
		- Eger nesnelerimiz 3 boyutlu olsaydi homography matrix degil de fundamental matrix bulurduk. (Spesifik olarak eger fotograflari ceken kameralari biliyorsak yani kalibre ettiysek essential matrix bulunmasi yeterli oluyor.) Bu konuya da gireriz ama zaten iki konu buyuk olcude kesisiyor.
	- Var olan algoritmalar bu problemi genelde iyi cozuyorlar. Bizim amacimiz sureci hizlandirmak.
- **A note on image matching vs. image retrieval**: Genel kultur niyetine... Image matching muhtemelen ayni sahneye ait olan iki fotograftaki noktalari eslestirmek. Bunu mesela image retrieval icin de kullanmak teoride mumkun. Ornegin Google'in imge ile arama ozelligi var. Eyfel kulesinin bir fotografini yuklersem sistemdeki tum imgelerle eslestirmeye calisip en iyi eslesenleri getirebilir. Ama bu bayagi naive olur. Imkansiz. Cunku veri tabaninda acayip fazla imge var. O yuzden image matching eslesme olasiligi yuksek olan imgelerin noktalarini eslestirmek olarak dusunulurken image retrieval ise eslesme olasiligi dusuk olan imgeleri (noktalarini degil! imgeleri) eslestirmek olarak dusunulebilir. Image matching icin noktalari (keypoint diyoruz) birer vektorle tanimlarken image retrieval icin mecburen (hiz icin) butun imgeyi tek bir vektorle tanimliyoruz. Birbirine benzer konular. Image matching samanlikta igne aramaksa image retrieval dunyada samanlik aramak gibi. Hem benzer problemler ve hem de beraber kullanilip birbirlerini tamamlayabiliyorlar cesitli uygulamalarda (Ornegin image matching yapacagim ama elimde binlerce image var. O zaman her ikiliyi tek tek match etmektense birbirine benzer olanlari bulurum, sonra sadece o pairleri match ederim).
# (1) Introduction – a Tour of Multiple View Geometry
## (1.1) Introduction – the ubiquitous projective geometry
- (Intro)
	- (Intro)
		- (**Projective transformation**)
			- When we look at a picture, we see squares that are not squares, or circles that are not circles. **The transformation that maps planar objects onto the picture is an example of a projective transformation.**
			- **What properties of geometry are preserved by projective transformations?**
				- Shapes: No (a circle may appear as an ellipse)
				- Lengths: No (two perpendicular radii of a circle are stretched by different amounts)
				- Angles, distance, ratios of distances: No
				- **Straightness**: Yes! (Duz bir cizgi yine duz bir cizgi olarak gozukur!)
			- (Definition) **Projective transformation of a plane**: 
				- **any mapping of the points on the plane that preserves straight lines.**
		- (**Projective geometry**, **projective space**, **points at infinity**)
			- Why do we need **projective geometry**?
				- Euclidean geometry is the geometry that describes angles and shapes of objects. Euclidean geometry is troublesome in one major respect – we need to keep making an exception to reason about some of the basic concepts of the geometry – such as intersection of lines. Two lines (we are thinking here of 2-dimensional geometry) almost always meet in a point, but there are some pairs of lines that do not do so – those that we call parallel. A common linguistic device for getting around this is to say that parallel lines meet “at infinity”. However this is not altogether convincing, and conflicts with another dictum, that infinity does not exist, and is only a convenient fiction. We can get around this by enhancing the Euclidean plane by the addition of these **points at infinity** where parallel lines meet, and resolving the difficulty with infinity by calling them “ideal points.”
			- **Euclidean space + points at infinity = Projective space**
				- By adding these points at infinity, the familiar Euclidean space is transformed into a new type of geometric object, projective space. This is a very useful way of thinking, since we are familiar with the properties of Euclidean space, involving concepts such as distances, angles, points, lines and incidence.
				- (Definition) **Projective space**:
					- **an extension of Euclidean space in which two lines always meet in a point, though sometimes at mysterious points at infinity.**
	- Coordinates
		- 2-dimensional Euclidean space: $\mathbb{R}^2$ (Bildigimiz 2 boyutlu uzay) 
			- Bu uzayda bir nokta: $(x, y)$
				- $(x_1, y_1) = (x_2, y_2)$ iff $x_1 = x_2$ and $y_1 == y_2$
				- Yani iki noktanin ayni veya farkli olmasi bilesenlerin ayni veya farkli olmasina bagli
			- (Oklit uzayi oldugu surece 3 boyutlu uzay vb. de olsa ayni mantik gecerli. Bu sefer $\mathbb{R}^3$ olur. Bizim $\mathbb{R}^3$ ile bir isimiz yok.)
		-  **2-dimensional projective space: $\mathbb{P}^2$** 
			- **Bu uzayda $\mathbb{R}^2$ var ve ayrica points at infinity var.** 
			- Points at infinity sonsuza uzanan bu duzlemde her yonde sonsuzdaki nokta olarak dusunulebilir (Bir bakima sonsuz yaricapli cember gibi).
			- Tuhaf ama bu points at infinity aslinda bir dogru! **Line at infinity** deniyor buna. (Dikkat, lines degil, 1 tane line.) 
			- ($\mathbb{P}^3$'te ise $\mathbb{R}^3$ ve lines at infinity var. Bu lines at infinity aslinda bir duzlem olusturuyor yani plane at infinity. Dikkat, planes degil, plane. Bizim $\mathbb{P}^3$ ile bir isimiz yok.) 
			- Bu uzaydaki noktalar Euclidean coordinates ile gosterilemez (sonsuzlugu da icerdigi icin), **homogeneous coordinates** ile gosterilir.
				- $\mathbb{P}^2$'deki bir nokta $(x, y, w)$ seklinde gosterilir. Bunu $\mathbb{R}^3$ ile karistirmamak lazim. Ayni notasyon kullaniliyor ama bu bir homojen koordinat diye basinda yazmak lazim. $(10, 20, 2)$ bir Oklit koordinati ise bu nokta sadece $(10, 20, 2)$ ile ayni noktadir. Halbuki homojen koordinat ise mesela $(5, 10, 1)$ de ayni noktadir. Yani $(10, 20, 2) = (5, 10, 1)$ ama tabii bunlar homojen koordinatlar ise! Mantigi asagida.
				- (Benzer sekilde $\mathbb{P}^3$'teki bir nokta $(x, y, z, w)$ diye gosterilir ama bizi ilgilendirmiyor. Normalde boyle bir kural tabii yok ama bizim icin su gecerli: Ne zaman 3 sayi gorursen homojen koordinatlar oldugunu, 2 sayi gorursen de Oklit koordinatlari oldugunu dusunebilirsin.)
				- **Conversion from $\mathbb{R}^2$ to $\mathbb{P}^2$**:
					- Sona $1$ ekle: **$(x, y)$ --> $(x, y, 1)$**
						- Ornegin $(2, 5)$ noktasini homojen koordinatlar ile $(2, 5, 1)$ ile gosteririz.
					- Alternatif olarak sona $0$'dan farkli herhangi bir $w$ sayisi ekleyebilirsin, ama $x$ ve $y$'yi de o sayiyla carpmalisin: $(2w, 5w, w)$. ($w=1$ alirsan zaten ustteki ile ayni oluyor.) ($w=0$ almak yasak, cunku her nokta icin $(0, 0, 0)$ elde edilecegi icin bilgi kaybedersin, sonra geri donusturemezsin koordinatlari.)
				- **Conversion from $\mathbb{P}^2$ to $\mathbb{R}^2$**:
					- Sonda $1$ varsa kaldir gitsin. Yani $(x, y, 1)$ --> $(x, y)$. 
						- Ornegin $(2, 5, 1)$ ise $(2, 5)$ olur.
					- Genel olarak: **$(x, y, w)$ --> $(x/y, y/w)$**
					- Simdi yukarida verilen homojen koordinatlarda saglanan $(10, 20, 2) = (5, 10, 1)$ esitliginin sebebini anlayabilirsin. Aslinda ikisi de bildigimiz $(5, 10)$.
					- Dikkat edersen $(x, y, w)$ --> $(x/y, y/w)$ formulunu $w=0$ iken uygulayamazsin. Bu tarz noktalar sonsuzdaki noktalardir ve $\mathbb{P}^3$'te olmalarina ragmen $\mathbb{R}^2$'de yoklar!
				- Ozet
					- **$(0, 0, 0)$ diye bir nokta yok.** 
						- Gecersiz. Yasak!
					- Ustteki haric **$(x, y, 0)$ seklindeki noktalarin her biri sonsuzdaki bir noktadir**. 
						- Bunlari $\mathbb{R}^2$'ye ceviremezsin. o uzayin bir elemani degil. Bu noktalarin her biri birbirinden farkli bir noktadir. Ve aslinda bu noktalar garip bir sekilde dogru olustururlar. (Buna daha sonra deginecegiz)
					- Usttekiler haric $(x, y, w)$ seklindeki bir nokta aslinda bildigin $(x/w, y/w)$ noktasidir.
					- Dolayisiyla $(x_1, y_1, w_1)$ ile $(x_2, y_2, w_2)$ ayni noktalar midir diye soruldugu zaman sunu yapmak lazim:
						- (Herhangi biri $(0, 0, 0)$ ise hata: Boyle bir nokta yok!)
						- Biri $0$ ile bitip digeri baska sayi ile bitiyorsa False. (Biri sonsuzdaki bir noktadir biri normal bir noktadir. Tabii ki ayni nokta degiller.)
						- Ikisi de $0$ ile bitiyorsa yani ikisi de sonsuzdaki bir nokta ise bu sefer sadece $x$'lere ve $y$'lere bakarak karar verilir. Yani $(x_1, y_1)$ esit midir $(x_2, y_2)$? (Yani aslinda sonsuzdaki noktalarin degree of freedom'u 2 gibi gozukuyor. Sonucta bir duzlemdeki her yondeki sonsuzlugu kapsiyor, degil mi? Yani sonsuz carpi sonsuz tane sonsuzda nokta var gibi dusunulebilir.)
						- Ikisi de $0$ ile bitmiyorsa (yani ikisi de sonlu noktalarsa), Oklit koordinatlarina cevirip karsilastirirsiz yani $(x_1/w_1, y_1/w_1)$ esit midir $(x_2/w_2, y_2/w_2)$? (Mesela belirttigimiz gibi $(10, 20, 2)$ ile $(5, 10, 1)$ ayni noktalar.)
	- Butun bunlar ne demek? Computer vision ile ne alakasi var?
		- Fotograflar uzerinden anlatalim. Zaten konumuz bu.
		- Dunyanin duz oldugunu ve sonsuz oldugunu varsay (zaten herhangi bir noktadan fotograf cekince dunyanin o kadar kucuk bir kismi cikiyor ki dunya duzmus gibi gozukuyor ve sonsuza kadar gidip gitmemesinin bir onemi de olmuyor). Ayrica dunyadaki butun nesneler de duzlemsel olsun. Biz butun projede duzlemsel nesnelerle ilgileniyoruz (duvar, kitap kapagi, resim vb.) Ama bir nesne 3 boyutlu bile olsa yeterince uzaktan bakinca kabaca duzlemsel kabul edilebilir. Cunku nesnenin cikinti ve girintilari bizim ona olan mesafemizin yaninda kucuk kalir. Dolayisiyla uzaktan baktigimiz bir binanin cephesini, pencereleri vb. 3 boyutlu olmasina ragmen duzlemsel sayabiliriz.
		- Ne demistik? Dunya bir duzlem (duz ve sonsuz). Ustundeki nesneler de duzlemsel (sonlu ya da sonsuz olabilir). Anlatmayi ve anlamayi kolaylastirmak icin bu nesnelerin hep yerde yatili olarak durduklarini dusunelim. Yani mesela dik duran duvarlarla degil yerde uzayan tren raylari falan var. Belki bu dunyayi absurt bulabilirsin ama aslinda bizim dunyamiza cok benzer. Dunyamiz zaten pratik olarak duzlem gibi. Duzlemsel olmayan nesneler de bizim calisma alanimiz degil, o yuzden onlari dunyadan sildik gitti (bizim fotograflarimizda yoklar veya varsalar uzakta olduklari icin duzlem gibi gozukuyorlar). Nesnelerin yere yapisik olmasiysa sadece simdilik anlamayi kolaylastirmak icin, gercek problemimizde oyle bir zorunluluk yok.
		- Bu dunyanin herhangi bir noktasina gidiyoruz. Yerde tren rayi oldugunu dusun (iki paralel dogru). Raylarin cok uzun oldugunu, hatta sonsuza kadar uzadigini varsay. Bulundugumuz noktadan basit bir (noktasal) kamera ile fotograf cekiyoruz. (Kameranin extrinsic parametreleri ne olursa olsun, fark etmez. Yani yerden yuksekligimiz, bakis acimiz vb. fark etmez.)
		- Fotografta cok tuhaf bir sey oluyor. Tren raylari paralel olduklari icin hic bulusmamasina (veya sonsuzda bulusmasina) ragmen fotografta sonlu bir yerde bulusuyorlar. Yani **projective transformation sayesinde sonsuzlugu (koordinat olarak) ayagimiza getiriyoruz!** (Bu kamerayla ilgili bir sey degil, gozumuzle bakarken veya resmini cizince de ayni sey soz konusu.) Cozunurlugumuz sonsuz olmadigi icin (ve tabii havada toz vb. oldugu icin) sonsuz mesafeyi goremiyoruz ama gorseydik kadrajimiza sigardi! Koordinatlarinin nereye denk geldigini biliyoruz! 
			- **Vanishing point** and **horizon** falan diye Google'layabilirsin! Bu paralel noktalarin bulustugu yerlere yani sonsuzdaki noktalara vanishing point diyoruz.
			- Kameranin bir duzlemin fotografini cekmesi aslinda bir **projective transformation**.
			- Bizim bakis acimizla sonsuzdaki noktalar sonlu bir noktaya donustu (Fotografi tepeden tam yere dogru bakarak cekersek donusmeyebilir tabii). Yani sonsuzdaki noktalar hayatimizin bir parcasi olabiliyorlar. Iste bu yuzden Euclidean space bize yetmiyor. Sonsuzdaki noktalari da kapsayan **projective space** gerekiyor.
			- Sonsuz farkli vanishing point vardir. Aslinda her paralel dogru kumesinin kendi vanishing pointi vardir. Yani birbirine paralel olan $5$ tane dogrumuz (veya dogru parcamiz) olsun. Hepsi ayni vanishing pointte bulusur. (Bakis acimiza bagli olarak bu vanishing point kadrajimiza girmeyebilir. Hatta tam dik acidan bakarsak bu paralel dogrulari birbirine paralel goruruz, yani kadrajimiza girmesi zaten imkansiz olur, sonsuzluk yine sonsuzda kalir. Ama onun disindaki tum acilarda bu paralel dogrulari paralel degillermis gibi goruruz. Projective transformation duz cizgileri duz tutar ama paralel cizgilerin paralelligini korumayabilir.) Yerde uzanan bu $5$ paralel dogruya paralel olmayan ama kendi aralarinda paralel baska $7$ tane dogru olsun. Bunlarin da kendilerine ozgu bir vanishing pointi olur.
			- Resimlerde gordugun horizon dedigimiz sey ise bu vanishing pointleri birlestiren dogrudur. Bu noktalarin bir dogru olusturdugunu soylemistim! (Sezgilerimiz bunun bir cember olmasi gerektigini soylese bile.)
			- Point at infinity = Vanishing point
				- Bunlardan sonsuz tane vardir. Herhangi birini 2 sayi soyleyerek tanimlayabiliriz. (Cunku duzlemde yon belirtmek icin 2 sayi soylemek gerekir.) Bu da aslinda $(x, y, 0)$ seklindeki koordinatlardaki $x$ ve $y$ sayilaridir.
			- Points at infinity (tekil degil, cogul! yani birlesim kumesi) = Line at infinity = Horizon (ufuk cizgisi)
		- Yukariyi anlamadan devam etme! Duzlemsel nesnelerimizin illa dunyanin ustune serili (tren rayi gibi yerde uzanir) olmasi gerekmez demistik (zaten gercek hayatta genelde boyle olmaz). Google'da gordugun resimlerden anlasilabilecegi gibi aslinda duzlemsel nesnelerin yere yapisik olup olmamasinin bir onemi yok. Onemli olan yer dedigimiz duzleme paralel olmalari! Mesela dik duran bir kutunun sadece yerde yatan alt yuzeyi degil, ust yuzeyi de yer duzlemine paraleldir. Onlar icin de vanishing point vardir ve hatta yerde serili olanlarinkiyle ayni noktalardir. ("Yerde yatan paralel dogru kumelerinin her birinin kendine has vanishing pointi vardir" diye ifade edebilecegimiz bilgiyi genellestirebiliriz artik: Kusbakisi bakinca gordugumuz paralel dogru kumelerinin (ister yerde yatsin ister yatmasin, yeter ki yer duzlemine paralel uzansin) kendine ozgu vanishing pointi vardir. Yani bir kutunun alt yuzeyinin dogrulari farkli yere gider, ust yuzeyinin dogrulari farkli yere gider degil! Bunlar ayni noktaya giderler! Resim cizerken onemli bir bilgidir bu, perspektif diye gecer.).
			- Ozet: Bizim duzlemlerimiz yere yatmak zorunda degil. Yere paralel olup yerden yuksek de olabilir.
		- Yukariyi anlamadan devam etme! Peki farkli duzlemler icin ne soyleyebiliriz? Yer duzlemine paralel olmayan duzlemler? Mesela ayakta duran bir kutunun (veya binanin) alti ve ustu degil de yanlari. Yani asagi-yukari ekseninde uzanan paralel dogrulari (veya dogru parcalari). Onlarin da kendi duzlemlerine has vanishing pointleri ve dolayisiyla horizonlari vardir. Fotografta gozukup gozukmemesi tamamen kameranin acisina bagli. Biz bu projede tek bir duzlem kumesiyle ilgileniyoruz (yere paralel ya da degil!), dolayisiyla bir tane ufuk cizgimiz var (genelde de kadraja sigmamis oluyor).
			- Ozet: Bizim duzlemlerimiz yere serili olmak zorunda degil demistik, yere paralel olmak zorunda da degil. Yeter ki tum duzlemlerimiz birbirine paralel olsun. (Ornegin bir binaya caprazdan bakip 2 ayri cephesini gormemeliyiz. Karsidan bakip birden fazla binanin ayni cephesini gorebiliriz. Biri bize daha yakin da olabilir. Yeter ki paralel duzlemler olsunlar.) Zaten bizim projemizde tek bir duzlem oluyor fotograflarimizda. Yani tek bir binanin tek bir cephesi gibi.
		- Duzlemsel bir nesnenin (veya birden fazla nesnenin, ama bu duzlemler birbirine paralel olmali!) fotografini cektigimiz zaman nesnenin fotografta nasil gozukecegi projective transformation ile hesaplaniliyor ve bununla ilgilenen geometri dali projective geometry.
		- Spoiler: Bir projective transformation'a ait parametreler (ornegin kameranin nesneye mesafesi, bakis acisi vb.) **homography** adi verilen bir matriste saklanir. (2 boyutlu nesnelerle degil de 3 boyutlu nesnelerle calisirsan homography matrix degil de fundamental matrix buluyorsun. Algoritmalarin neredeyse tamami orada da gecerli.)
	- Homogeneity
		- **(Homogeneity of points in Euclidean space)** 
			- In classical Euclidean geometry **all points are the same**. (There is no distinguished point. The whole of the space is homogeneous.) When coordinates are added, one point is seemingly picked out as the origin.
		- **(Euclidean transform)** 
			- We can consider a change of coordinates for the Euclidean space in which the axes are shifted and rotated to a different position. We may think of this in another way as the space itself **translating** and **rotating** to a different position. The resulting operation is known as a Euclidean transform.
		- **(Affine transform)** 
			- A more general type of transformation is that of applying a **linear transformation to $\mathbb{R}^n$, followed by a Euclidean transformation** moving the origin of the space. We may think of this as the space **moving**, **rotating** and finally **stretching linearly possibly by different ratios in different directions**. The resulting transformation is known as an affine transformation.
		- **(Points at infinity remains at infinity)**
			- The result of either a **Euclidean or an affine transformation** is that **points at infinity remain at infinity**.
		-  **(Homogeneity of points in projective space)**
			- From the point of view of **projective geometry**, **points at infinity are not any different from other points**. Just as Euclidean space is uniform, so is projective space.
		- **(Projective transform; points at infinity do not remain at infinity)** 
			- By analogy with Euclidean or affine transformations, we may define a **projective transformation** of projective space. A linear transformation of Euclidean space $\mathbb{R}^n$ is represented by matrix multiplication applied to the coordinates of the point. In just the same way a projective transformation of projective space $\mathbb{R}^n$ is a mapping of the homogeneous coordinates representing a point (an $(n + 1)$-vector), in which the coordinate vector is multiplied by a non-singular matrix. Under such a mapping, **points at infinity (with final coordinate zero) are mapped to arbitrary other points.** The points at infinity are not preserved. Thus, **a projective transformation of projective space $\mathbb{R}^n$ is represented by a linear transformation of homogeneous coordinates $X' = H_{(n+1)\times(n+1)}X$.**
				- Biz $\mathbb{R}^2$ ile ilgilendigimiz icin bizim $H$'lerimiz $3 \times 3$ boyutunda. Matris carpiminin gerceklesebilmesi icin elbette $X$'lerimiz 3 elemanli vektorler (ki onlar da bizim homojen koordinatlarimiz).
- (1.1.1) Affine and Euclidean Geometry
	- Affine geometry
		- (Asiri onemli degil.)
	- Euclidean geometry
		- (Asiri onemli degil.)
# (2) Projective Geometry and Transformations of 2D
## (2.1) Planar geometry
* (Asiri onemli degil.)
## (2.2) The 2D projective plane
* Homogeneous representation of lines.
	* ...
* Homogeneous representation of points.
	* ...
* Result 2.1.
	* ...
* Intersection of lines and Result 2.2.
	* ...
* Line joining points and Result 2.4.
	* ...
* Ideal points and the line at infinity.
	* ...
* (Devami asiri onemli degil.)
## (2.3) Projective transformations
- (Intro)
	- ...
- Definition 2.9.
	- ...
- Theorem 2.10.
	- ...
- Definition 2.11. Projective transformation.
	- ...
- (Devami asiri onemli degil.)
## (2.4) A hierarchy of transformations
# (4) Estimation – 2D Projective Transformations
## (4.1) The Direct Linear Transformation (DLT) algorithm
## (4.2) Different cost functions
## (4.4) Transformation invariance and normalization
- Sadece Algorithm 4.2
## (4.5) Iterative minimization methods
- Sadece Algorithm 4.3
## (4.7) Robust estimation
 Tamamı (Algoritma olarak hepsi: Algorithm 4.4 & Algorithm 4.5 & Algorithm 4.6)
## (4.8) Automatic computation of a homography
Tamamı
