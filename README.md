
![lugatitturk-high-resolution-logo (3)](https://github.com/user-attachments/assets/dba710a6-e246-44fb-872d-78ead64bd9ac)

# Teknofest 2024 NLP (Türkçe Doğal Dil İşleme) Yarışması Github Projesi


## Modellerin Kullanımı
Projemizde 2 modeli birleştirerek tek bir model oluşturduk. Bu alt modellerden LügatitBert girdiyi(leri) sınıflandırma görevini yerine getirirken LugatitHaberler girdinin temasını belirlemektedir. LügatitBert girdiyi üç sınıfa ayırıyor, bunlar şiddet içerikli (zorbalık, küfür, hakaret vb.), yönlendirici (toplumu eylem, toplumsal hareket, galeyan gibi herhangi bir aksiyon almaya yönlendiren/ teşvik eden) ve diğer. LugatitHaberler girdiyi yedi temaya ayırıyor, bunlar Dünya, Ekonomi, Kültür, Sağlık, Siyaset, Spor, Diğer. Oluşturduğumuz bu model iki çeşit girdi alabiliyor ve bu girdinin tipine göre çıktı veriyor. 

**Kullanım seneryosu 1:** Tekil girdi
Modelimize tek bir tweet/cümle/metin girdi olarak verildiğinde öncelikle LügatitBert bu tekil girdinin sınflandırmasını yapıyor. Eğer sınıflandırma sonucu ana odağımız olan 'yönlendirici' kategorisi olursa bu girdi hakkında daha fazla bilgi edinmek amacıyla LugatitHaberler girdinin temasını belirliyor. Bu durumda LügatitBert her sınıf için bir aidiyet olasılığı çıktı veriyor ve girdi, aidiyet olasılığının en yüksek olduğu sınıfa atanıyor. LugatitHaberler de aynı şekilde her tema için bir aidiyet olasılığı çıktı veriyor ve girdi, aidiyet olasılığının en yüksek olduğu temaya atanıyor. 

**Kullanım seneryosu 2:** Çoklu girdi
Modelimize birden fazla tweet/cümle/metin excel dosyası formatında girdi olarak verildiğinde öncelikle LügatitBert sınıfların exceldeki dağılımını çıktı veriyor. Eğer girilen excel 'yönlendirici' sınıfında veri içeriyosa LugatitHaberler ile bu sınıfta olduğu belirlenen verilerin tema dağılımı çıktı veriliyor. Sınıflar ve temalar belirlenirken yine aidiyet olasılıklarından en yükseği kullanılıyor.


### 1) Hugging Face Aracılığıyla
Tüm denediğimiz modeller arasından **en iyi 5 modeli** HuggingFace sayfamıza (https://huggingface.co/LugatitTurk) yüklemiş bulunmaktayız. Bu sayede modelimizi denemek veya kullanmak için githubdan modeli indirmeniz gerekmemektedir. 
HuggingFace aracılığıyla modeli kullanmak için,

```
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import torch
import torch.nn.functional as F

egilimTokenizer = AutoTokenizer.from_pretrained("LugatitTurk/LugatitBert")
egilimModel = AutoModelForSequenceClassification.from_pretrained("LugatitTurk/LugatitBert")

haberTokenizer = AutoTokenizer.from_pretrained("LugatitTurk/LugatitHaberler")
haberModel = AutoModelForSequenceClassification.from_pretrained("LugatitTurk/LugatitHaberler")


```
```
def MetinTahmini(metin):
  inputs = egilimTokenizer(metin, return_tensors="pt")
  logits = egilimModel(**inputs)
  outputs = logits[0][0][:3]
  probabilities = F.softmax(outputs, dim=0)
  percentages = probabilities * 100
  result = percentages.detach().numpy()

  if( result[0] < result[2] and result[1] < result[2] ):
    inputs = haberTokenizer(metin, return_tensors="pt")
    logits = haberModel(**inputs)
    outputs = logits[0][0]
    probabilities = F.softmax(outputs, dim=0)
    percentages = probabilities * 100
    result2 = percentages.detach().numpy()
    return [result,result2]
  else :
     return result

```
yöntemini kullanabilirsiniz.

### 2) Github'daki modelin dosyalarıyla
Github'da bulunan modelin boyutu 100mb'dan büyük olduğundan ötürü model **git-lfs** yönetemiyle yüklenmiştir. Modeli kullanmak için;

1. Git LFS'i Kur:
Eğer bilgisayarında Git LFS yüklü değilse, öncelikle Git LFS'i kurman gerekir. Terminale şu komutu yazarak kurabilirsin:
```
git lfs install
```

2. Depoyu Klonla:
Depoyu klonladığında, Git LFS tarafından izlenen büyük dosyalar otomatik olarak indirilecektir. Depoyu klonlamak için;
```
git clone https://github.com/LugatitTurk/Teknofest_TDI.git
```

3. Modeli Kullanma
Modeli clone'ladıktan sonra modeli kullanabilirsiniz. Modeli kullanmak için;
```
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import torch
import torch.nn.functional as F

egilimTokenizer = AutoTokenizer.from_pretrained("LugatitBert modelinin konumu")
egilimModel = AutoModelForSequenceClassification.from_pretrained("LugatitBert modelinin konumu")

haberTokenizer = AutoTokenizer.from_pretrained("LugatitHaberler modelinin konumu")
haberModel = AutoModelForSequenceClassification.from_pretrained("LugatitHaberler modelinin konumu")

```
```
def MetinTahmini(metin):
  inputs = egilimTokenizer(metin, return_tensors="pt")
  logits = egilimModel(**inputs)
  outputs = logits[0][0][:3]
  probabilities = F.softmax(outputs, dim=0)
  percentages = probabilities * 100
  result = percentages.detach().numpy()

  if( result[0] < result[2] and result[1] < result[2] ):
    inputs = haberTokenizer(metin, return_tensors="pt")
    logits = haberModel(**inputs)
    outputs = logits[0][0]
    probabilities = F.softmax(outputs, dim=0)
    percentages = probabilities * 100
    result2 = percentages.detach().numpy()
    return [result,result2]
  else :
     return result
```
yöntemini kullanabilirsiniz.


## Datasetleri 
Projemiz 2 model içermektedir. Bu modellerin eğitimi için kullanılan datasetleri data klasörünün içindeki datasets klasöründe bulunmaktadır.

#### LügatitBert'in Datasetleri
LugatiBert modelimizi eğitmek için;
  - data klasöründe bulunan data scraper kodu aracılığıyla Twitter'dan elde edilen veriler,
    
  - Hazır datasetleri
    - https://huggingface.co/datasets/Overfit-GM/turkish-toxic-language
    - https://huggingface.co/datasets/nanelimon/insult-dataset
      
  - Sentetik veriler

kullanılmıştır.

#### LugatitHaberler'in Datasetleri
LugatiHaber modelimizi eğitmek için;
  - Hazır dataseti
      - https://www.kaggle.com/datasets/anil1055/turkish-headlines-dataset

kullanılmıştır.



![Interactive-Neural-Network-Opera-2024-08-07-01-03-28-_online-video-cutter com_](https://github.com/user-attachments/assets/cf4772c9-90a8-4251-8106-092f19cf472f)





