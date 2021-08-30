---
title: "Cordova Eklentisi Oluşturma"
date: 2021-08-30T21:26:18+03:00
draft: true
---

![Cordova eklentisi](/img/cordova-plugin.png)

Bu yazı ile basit bir Cordova eklentisinin nasıl oluşturulacağı, ne olduğu ve ne zaman ihtiyaç duyduğumuzu anlatmaya çalışacağım.
### Cordova Eklentisi Nedir?

Başlamadan önce Cordova eklentilerinin ne olduğunu anlayalım. Cordova eklentisi web ve native sistem arasında köprü işlevi görür. Biraz JavaScript biraz da native kod ile bu görevi üstlenir.

### Ne Zaman İhtiyaç Duyarız?

Cordova uygulamaları temelde bir web tarayıcısında çalışır. Bununla birlikte web her zaman native sistemde ihtiyaç duyacağımız özellikleri sağlamaz. Bu sebeple native kod yazıp sistem ile iletişime geçmemiz gerekiyor. Cordova eklentisi bunu sağlar.

### Nasıl Oluşturulur?

Eklenti oluşturmak için öncelikle Cordova CLI ve Plugman'a ihtiyacımız var. Aşağıdaki komut ile terminalden kurulum yapabilirsiniz.

{{< highlight html >}}
npm install -g plugman cordova
{{< /highlight >}}

Ardından Plugman aracılığıyla eklentiyi oluşturabiliriz. Toplam 3 parametre belirtmemiz gerekiyor.

{{< highlight html >}}
plugman create --name PluginName --plugin_id "me.mazlum.pluginname" --plugin_version 0.0.1
{{< /highlight >}}

Komutu çalıştırdıktan sonra bulunduğunuz dizinde pluginin temel klasör ve dosyalarını oluşturacaktır. Daha sonra oluşturulan klasörün içerisine girip aşağıdaki komut ile plugine platform ekleyebiliriz. Örneğin android üzerinden gidelim.

{{< highlight html >}}
plugman platform add --platform_name android
{{< /highlight >}}

Yukarıdaki komut çalıştırıldıktan sonra örnek bir Java dosyanız oluşacak. Android platformu için yazacağınız tüm kodlar bu dosya üzerinden yönetilecek. Buraya kadar geldiğiniz durumda aslında elinizde çok basit bir eklenti oluyor. Eğer bu eklentiyi NPM ile yayınlamak isterseniz, aşağıdaki komut ile package.json dosyasını oluşturabilirsiniz.

{{< highlight html >}}
plugman createpackagejson .
{{< /highlight >}}

> Bu kısım itibariyle örnekler Android üzerine olacak.

### JavaScript API

Yazının başlangıcında bahsettiğim gibi biraz JavaScript biraz da native kod demiştik. İlk önce JavaScript tarafında çalışacağımız kısma bir göz atalım. Aşağıdaki dosyaya **www/PluginName.js** dizininden ulaşabilirsiniz.

{{< highlight javascript >}}
var exec = require('cordova/exec');
exports.coolMethod = function (arg0, success, error) {
  exec(success, error, 'PluginName', 'coolMethod', [arg0]);
};
{{< /highlight >}}

Bu dosyada oluşturulan fonksiyonlar aracılığıyla native kod ile iletişim kuracağız. Örneğin JavaScript tarafında bu fonksiyonu aşağıdaki şekilde çağırabiliriz.

{{< highlight javascript >}}
cordova.plugins.PluginName.coolMethod('test', (success) => {
  console.log('response', success);
});
{{< /highlight >}}

Bir de bunun native taraftaki karşılığına bakalım. Aşağıdaki dosyaya **src/android/PluginName.java** dizininden ulaşabilirsiniz.

{{< highlight java >}}
package me.mazlum.pluginname;
import org.apache.cordova.CordovaPlugin;
import org.apache.cordova.CallbackContext;
import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

public class PluginName extends CordovaPlugin {
  @Override
  public boolean execute(String action, JSONArray args, CallbackContext callbackContext) throws JSONException {
    if (action.equals("coolMethod")) {
      String message = args.getString(0);
      this.coolMethod(message, callbackContext);
      return true;
    }

    return false;
  }

  private void coolMethod(String message, CallbackContext callbackContext) {
    if (message != null && message.length() > 0) {
      callbackContext.success(message);
    } else {
      callbackContext.error("Expected one non-empty string argument.");
    }
}
{{< /highlight >}}

**execute** adlı metot aracılığıyla JavaScript tarafından gönderilen parametreleri yakalayıp işliyoruz. Yukarıdaki örnekte **coolMethod** güzel bir örnek.

### Eklentiye Dependency Ekleme

Plugin oluşturduktan sonra elbet başka bir pakete ihtiyacınız olacaktır. Peki bu durumda Cordova eklentinize nasıl paket ekleyeceksiniz? Yapmamız gereken işlem basit. CLI ile eklentiyi oluştururken **plugin.xml** adında bir dosya da oluşturuldu. Bu dosya içerisine paketlerimizi ekleyebiliriz. Örneğin Agora Voice SDK import edelim.

{{< highlight xml >}}
<platform name="android">
  ....
  <framework src="io.agora.rtc:voice-sdk:3.3.0" />
</platform>
{{< /highlight >}}

### Eklentiye Permission Ekleme

Örneğin mikrofon izinine ihtiyacımız olacak bu SDK'i kullanırken. O zaman eklenti kurulduğunda, **AndroidManifest.xml** dosyamıza bu izini aşağıdaki kod ile otomatik eklemesini sağlayalım. Yine **plugin.xml** dosyamızda düzenleme yapmalıyız.

{{< highlight xml >}}
<platform name="android">
  ....
  <config-file parent="/manifest" target="AndroidManifest.xml">
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
  </config-file>
</platform>
{{< /highlight >}}

### Eklentiye Değişkenler Ekleme

Bir eklentiyi kurarken sabit bir değişken isteyebiliriz. Bu değişkenler platform özelinde veya direkt genel olabilir. Örneğin **cordova-plugin-purchase** eklentisini kurarken **BILLING_KEY** adında bir değişken vermeniz gerekir. Parametre ile eklenti kurulumuna bir örnek.

{{< highlight html >}}
cordova plugin add cordova-plugin-purchase --variable BILLING_KEY="MIIB...AQAB"
{{< /highlight >}}

Peki bu değişkeni nerede tanımlıyoruz? Evet yine **plugin.xml**.

{{< highlight xml >}}
<preference name="MY_GLOBAL_VARIABLE" default=" " />
<platform name="android">
  ....
  <preference name="MY_VARIABLE" default=" " />
</platform>
{{< /highlight >}}

### Eklentinin Bir Projeye Kurulması

Eklentimizi oluşturduk, paketlerimizi ekledik, değişkenlerimizi tanımladık. Artık bu eklentiyi bir cordova projesinde kullanmaya hazırız. Eğer eklentiyi henüz NPM'de yayınlamadıysanız aşağıdaki şekilde cordova projenize ekleyebilirsiniz. Cordova projenizin ana dizinine gidip aşağıdaki komut ile eklentinizi projenize ekleyebilirsiniz.

{{< highlight html >}}
cordova plugin add --link ~/path/PluginName
{{< /highlight >}}

Artık bu aşamadan sonra dilediğiniz şekilde native kodda düzenleme yapabilirsiniz.

### Son

Bu yazı ile birlikte küçük bir Cordova eklentisi oluşturmayı ve ufak tefek özelleştirme yapmayı öğrendik. Bir sonraki yazımda görüşmek üzere.
