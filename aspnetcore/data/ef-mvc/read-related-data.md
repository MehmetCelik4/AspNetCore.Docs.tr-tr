---
title: 'Öğretici: EF Core ile ilgili verileri okuma-ASP.NET MVC'
description: Bu öğreticide ilgili verileri okuyabilir ve görüntüleyebilirsiniz. Bu, Entity Framework, gezinti özelliklerine yüklediği verileri okur.
author: rick-anderson
ms.author: riande
ms.date: 09/28/2019
ms.topic: tutorial
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: data/ef-mvc/read-related-data
ms.openlocfilehash: 7cc400218d7ad8717311e56ec5b3fc190857862c
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82773542"
---
# <a name="tutorial-read-related-data---aspnet-mvc-with-ef-core"></a>Öğretici: EF Core ile ilgili verileri okuma-ASP.NET MVC

Önceki öğreticide, okul veri modelini tamamladınız. Bu öğreticide ilgili verileri okur ve görüntüleriz. Bu, Entity Framework, gezinti özelliklerine yüklediği veriler.

Aşağıdaki çizimlerde, birlikte çalışacağımız sayfalar gösterilmektedir.

![Kurslar Dizin sayfası](read-related-data/_static/courses-index.png)

![Eğitmenler Dizin sayfası](read-related-data/_static/instructors-index.png)

Bu öğreticide şunları yaptınız:

> [!div class="checklist"]
> * İlgili verileri yüklemeyi öğrenin
> * Kurslar sayfası oluşturma
> * Eğitmenler sayfası oluşturma
> * Açık yükleme hakkında bilgi edinin

## <a name="prerequisites"></a>Önkoşullar

* [Karmaşık veri modeli oluşturma](complex-data-model.md)

## <a name="learn-how-to-load-related-data"></a>İlgili verileri yüklemeyi öğrenin

Entity Framework gibi nesne Ilişkisel eşleme (ORM) yazılımının bir varlığın gezinti özelliklerine ilgili verileri yükleyebilmesinin birkaç yolu vardır:

* Eager yükleniyor. Varlık okurken ilgili veriler onunla birlikte alınır. Bu, genellikle gereken tüm verileri alan tek bir JOIN sorgusuna neden olur. `Include` Ve `ThenInclude` yöntemlerini kullanarak Entity Framework Core ' de bir Eager yüklemesi belirlersiniz.

  ![Eager yükleme örneği](read-related-data/_static/eager-loading.png)

  Bazı verileri ayrı sorgularda alabilir ve "düzeltmeler" i "düzeltir" seçeneğini kullanabilirsiniz.  Diğer bir deyişle, EF, daha önce alınan varlıkların gezinti özelliklerine ait oldukları ayrı alınan varlıkları otomatik olarak ekler. İlgili verileri alan sorgu için, `Load` `ToList` veya `Single`gibi bir liste veya nesne döndüren bir yöntem yerine yöntemini kullanabilirsiniz.

  ![Ayrı sorgular örneği](read-related-data/_static/separate-queries.png)

* Açık yükleme. Varlık ilk kez okunmadıysa ilgili veriler alınmadı. Gerekirse ilgili verileri alan kodu yazarsınız. Ayrı sorgularla yükleme durumunda olduğu gibi, açıkça yükleme, veritabanına gönderilen birden çok sorgu ile sonuçlanır. Fark, açık yükleme ile kod, yüklenecek gezinti özelliklerini belirtir. Entity Framework Core 1,1 ' de, `Load` açık yükleme yapmak için yöntemini kullanabilirsiniz. Örneğin:

  ![Açık yükleme örneği](read-related-data/_static/explicit-loading.png)

* Yavaş yükleme. Varlık ilk kez okunmadıysa ilgili veriler alınmadı. Ancak, bir gezinti özelliğine ilk kez erişmeye çalıştığınızda, bu gezinti özelliği için gereken veriler otomatik olarak alınır. Bir gezinti özelliğinden ilk kez veri almaya çalıştığınızda veritabanına bir sorgu gönderilir. Entity Framework Core 1,0, yavaş yüklemeyi desteklemez.

### <a name="performance-considerations"></a>Performansla ilgili önemli noktalar

Alınan her varlık için ilgili verilerin gerekli olduğunu biliyorsanız, tek bir sorgu genellikle en iyi performansı sunar, çünkü veritabanına gönderilen tek bir sorgu genellikle alınan her varlık için ayrı sorgulardan daha etkilidir. Örneğin, her departmanın on ile ilgili kurs olduğunu varsayalım. Tüm ilgili verilerin bir şekilde yüklenmesi, tek bir (JOIN) sorgusuna ve veritabanına yönelik tek gidiş dönüş oluşmasına neden olur. Her bölüme yönelik kurslar için ayrı bir sorgu, veritabanı üzerinde on bir gidiş dönüş oluşmasına neden olur. Gecikme süresi yüksek olduğunda veritabanına yönelik ek gidiş dönüşler özellikle performansa neden olur.

Öte yandan, bazı senaryolarda ayrı sorgular daha etkilidir. Tek bir sorgudaki ilgili verilerin tümünü yükleme, SQL Server verimli bir şekilde bir birleştirmenin oluşturulmasına neden olabilir ve bu da etkili bir şekilde işleyemez. Ya da bir varlığın gezinti özelliklerine yalnızca işlemekte olduğunuz varlıkların kümesinin bir alt kümesi için erişmeniz gerekiyorsa, her şeyin en baştan yüklenmesi gerekenden fazla veri alacağından ayrı sorgular daha iyi çalışabilir. Performans önemliyse, en iyi seçimi yapmak için her iki şekilde de performansı test etmek en iyisidir.

## <a name="create-a-courses-page"></a>Kurslar sayfası oluşturma

Kurs varlığı, kursun atandığı departmanın departman varlığını içeren bir gezinti özelliği içerir. Bir kurs listesinde atanan departmanın adını göstermek için, `Course.Department` Gezinti özelliğindeki departman varlığındaki Name özelliğini almanız gerekir.

Aşağıdaki çizimde gösterildiği gibi, daha önce öğrenciler denetleyicisi için yaptığınız Entity Framework desteği ' ı kullanarak, **MVC denetleyicisi için, görünümler ile** aynı seçenekleri kullanarak kurs varlık türü için coursescontroller adlı bir denetleyici oluşturun:

![Kurs denetleyicisi ekleme](read-related-data/_static/add-courses-controller.png)

*CoursesController.cs* açın ve `Index` metodunu inceleyin. Otomatik yapı iskelesi, `Department` `Include` yöntemi kullanılarak gezinti özelliği için bir Eager yüklemesi belirtti.

`Index` Yöntemini `IQueryable` , kurs varlıklarını döndüren ' için daha uygun bir ad kullanan aşağıdaki kodla değiştirin (`courses` yerine `schoolContext`):

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_RevisedIndexMethod)]

*Views/kurslar/Index. cshtml* dosyasını açın ve şablon kodunu aşağıdaki kodla değiştirin. Değişiklikler vurgulanır:

[!code-html[](intro/samples/cu/Views/Courses/Index.cshtml?highlight=4,7,15-17,34-36,44)]

Yapı iskelesi kodunda aşağıdaki değişiklikleri yaptınız:

* Başlık dizinden kurslar olarak değiştirildi.

* Özellik değerini gösteren bir sayı sütunu eklendi. **Number** `CourseID` Birincil anahtarlar, genellikle son kullanıcılara anlamlı olduklarından, varsayılan olarak yapı iskelesi göstermemektedir. Ancak, bu durumda birincil anahtar anlamlı olur ve göstermek istersiniz.

* Departman adını göstermek için **Departman** sütunu değiştirildi. Kod, `Department` gezinti özelliğine `Name` yüklenen departman varlığının özelliğini görüntüler:

  ```html
  @Html.DisplayFor(modelItem => item.Department.Name)
  ```

Uygulamayı çalıştırın ve bölüm adlarıyla listeyi görmek için **Kurslar** sekmesini seçin.

![Kurslar Dizin sayfası](read-related-data/_static/courses-index.png)

## <a name="create-an-instructors-page"></a>Eğitmenler sayfası oluşturma

Bu bölümde, Eğitmenler sayfasını görüntülemek için eğitmen varlığı için bir denetleyici ve görünüm oluşturacaksınız:

![Eğitmenler Dizin sayfası](read-related-data/_static/instructors-index.png)

Bu sayfa aşağıdaki yollarla ilgili verileri okur ve görüntüler:

* Eğitmenler listesi, OfficeAssignment varlığındaki ilgili verileri görüntüler. Eğitmen ve OfficeAssignment varlıkları bire sıfır veya-bir ilişkidir. OfficeAssignment varlıkları için Eager yükleme kullanacaksınız. Daha önce açıklandığı gibi, birincil tablonun alınan tüm satırları için ilgili verilere ihtiyacınız olduğunda Eager yüklemesi genellikle daha etkilidir. Bu durumda, tüm görüntülenen Eğitmenler için Office atamalarını göstermek istersiniz.

* Kullanıcı bir eğitmen seçtiğinde ilgili kurs varlıkları görüntülenir. Eğitmen ve kurs varlıkları çoktan çoğa bir ilişkidir. Kurs varlıkları ve ilgili departman varlıkları için Eager yükleme kullanacaksınız. Bu durumda, yalnızca seçili eğitmen için kurslar gerektiğinden ayrı sorgular daha verimli olabilir. Ancak, bu örnek, gezinti özellikleri ' nde olan varlıkların içindeki gezinti özellikleri için Eager yükleme 'nin nasıl kullanılacağını gösterir.

* Kullanıcı bir kurs seçtiğinde, kayıt varlığı kümesindeki ilgili veriler görüntülenir. Kurs ve kayıt varlıkları bire çok ilişkisinde. Kayıt varlıkları ve ilgili öğrenci varlıkları için ayrı sorgular kullanacaksınız.

### <a name="create-a-view-model-for-the-instructor-index-view"></a>Eğitmen dizini görünümü için bir görünüm modeli oluşturun

Eğitmenler sayfasında, üç farklı tablodan alınan veriler gösterilir. Bu nedenle, her biri tablolardan birine ait verileri tutan üç özellik içeren bir görünüm modeli oluşturacaksınız.

*SchoolViewModels* klasöründe *InstructorIndexData.cs* oluşturun ve mevcut kodu şu kodla değiştirin:

[!code-csharp[](intro/samples/cu/Models/SchoolViewModels/InstructorIndexData.cs)]

### <a name="create-the-instructor-controller-and-views"></a>Eğitmen denetleyicisi ve görünümleri oluşturma

Aşağıdaki çizimde gösterildiği gibi EF okuma/yazma eylemleri ile bir eğitmenler denetleyicisi oluşturun:

![Eğitmenler denetleyicisi Ekle](read-related-data/_static/add-instructors-controller.png)

*InstructorsController.cs* açın ve ViewModel ad alanı için bir using ifadesini ekleyin:

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_Using)]

İlişkili verilerin yüklenmesini ve görünüm modeline koymak için Dizin yöntemini aşağıdaki kodla değiştirin.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_EagerLoading)]

Yöntemi, seçilen eğitmenin ve seçili kursun`id`kimlik değerlerini sağlayan isteğe bağlı yol verilerini`courseID`() ve bir sorgu dizesi parametresini () kabul eder. Parametreler, sayfadaki Köprü **seçme** ile sağlanır.

Kod, görünüm modelinin bir örneğini oluşturarak ve bu örneğe eğitmen listesine yerleştirilerek başlar. Kod, `Instructor.OfficeAssignment` ve `Instructor.CourseAssignments` gezinti özellikleri için Eager yüklemeyi belirtir. `CourseAssignments` Özelliği içinde, `Course` özelliği yüklenir, ve içinde, `Enrollments` ve `Department` özellikleri yüklenir ve her `Enrollment` bir varlıkta `Student` özelliği yüklenir.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ThenInclude)]

Görünüm her zaman OfficeAssignment varlığını gerektirdiğinden, bunu aynı sorguda getirmek daha etkilidir. Web sayfasında bir eğitmen seçildiğinde kurs varlıkları gereklidir, bu yüzden tek bir sorgu birden çok sorgudan daha iyidir, bu nedenle yalnızca sayfa, hiçbir zaman tarafından seçilen bir kurs ile daha sık görüntüleniyorsa.

Kodu yinelenir `CourseAssignments` ve ' `Course` den `Course`iki özellik olması gerekir. İlk `ThenInclude` çağrı dizesi, `CourseAssignment.Course` `Course.Enrollments`ve `Enrollment.Student`' ı alır.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ThenInclude&highlight=3-6)]

Bu noktada, başka bir deyişle, ihtiyacınız `ThenInclude` olmayan, ' nin `Student`gezinti özellikleri için bir tane olur. Ancak çağıran `Include` özellikleri ile `Instructor` başlar, bu kez `Course.Department` zinciri yeniden gitmeniz gerekir. `Course.Enrollments`

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ThenInclude&highlight=7-9)]

Aşağıdaki kod, bir eğitmen seçildiğinde yürütülür. Seçilen eğitmen, görünüm modelindeki eğitmenler listesinden alınır. Görünüm modelinin `Courses` özelliği daha sonra bu eğitmenin `CourseAssignments` gezinti özelliğinden kurs varlıklarıyla birlikte yüklenir.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?range=56-62)]

`Where` Yöntemi bir koleksiyon döndürür, ancak bu yönteme geçirilen kriterler yalnızca tek bir eğitmen varlığının döndürüldüğünden sonuçlanır. `Single` Yöntemi, koleksiyonu tek bir eğitmen varlığına dönüştürür, bu da o varlığın `CourseAssignments` özelliğine erişmenizi sağlar. `CourseAssignments` Özelliği, yalnızca `CourseAssignment` ilgili `Course` varlıkları istediğiniz varlıkları içerir.

Koleksiyonda yalnızca bir `Single` öğe olacağını bildiğiniz durumlarda yöntemini kullanırsınız. Tek yöntem, koleksiyon boş veya birden fazla öğe varsa bir özel durum oluşturur. Koleksiyon boşsa varsayılan `SingleOrDefault`bir değer (Bu durumda null) döndüren alternatif bir alternatiftir. Bununla birlikte, bu durumda yine de bir özel durum oluşmasına neden olur (null başvuru üzerinde bir `Courses` Özellik bulmaya çalışırken) ve özel durum iletisi sorunun nedenini daha az göstermez. `Single` Yöntemini çağırdığınızda, `Where` yöntemi ayrı olarak çağırmak yerine WHERE koşulunu da geçirebilirsiniz:

```csharp
.Single(i => i.ID == id.Value)
```

Onun yerine:

```csharp
.Where(i => i.ID == id.Value).Single()
```

Ardından, bir kurs seçilmişse, seçilen kurs, görünüm modelindeki kurslar listesinden alınır. Ardından, görünüm modelinin `Enrollments` özelliği, bu kursun `Enrollments` gezinti özelliğinden kayıt varlıklarıyla birlikte yüklenir.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?range=64-69)]

### <a name="modify-the-instructor-index-view"></a>Eğitmen dizini görünümünü değiştirme

*Views/eğitmenler/Index. cshtml*içinde, şablon kodunu aşağıdaki kodla değiştirin. Değişiklikler vurgulanır.

[!code-html[](intro/samples/cu/Views/Instructors/Index1.cshtml?range=1-64&highlight=1,3-7,15-19,24,26-31,41-54,56)]

Varolan koda aşağıdaki değişiklikleri yaptınız:

* Model sınıfı olarak `InstructorIndexData`değiştirildi.

* Sayfa başlığı **dizinden** **eğitmenler**olarak değiştirildi.

* Yalnızca null olmaması halinde `item.OfficeAssignment` görüntülenen bir Office sütunu eklendi. **Office** `item.OfficeAssignment.Location` (Bu bire sıfır veya-bir ilişki olduğundan ilgili bir OfficeAssignment varlığı bulunmayabilir.)

  ```html
  @if (item.OfficeAssignment != null)
  {
      @item.OfficeAssignment.Location
  }
  ```

* Her bir eğitmen tarafından taders kurslarını görüntüleyen bir **Kurslar** sütunu eklendi. Daha fazla bilgi için, Razor söz dizimi makalesindeki [açık satır geçişi](xref:mvc/views/razor#explicit-line-transition) bölümüne bakın.

* Seçilen eğitmenin `tr` öğesine dinamik olarak `class="table-success"` ekleyen kod eklendi. Bu, bir önyükleme sınıfı kullanarak seçili satır için bir arka plan rengi ayarlar.

  ```html
  string selectedRow = "";
  if (item.ID == (int?)ViewData["InstructorID"])
  {
      selectedRow = "table-success";
  }
  <tr class="@selectedRow">
  ```

* Her satırdaki diğer bağlantılardan hemen önce **Seç** etiketli yeni bir köprü eklendiğinde, SEÇILEN eğitmenin kimliğinin `Index` yönteme gönderilmesine neden olur.

  ```html
  <a asp-action="Index" asp-route-id="@item.ID">Select</a> |
  ```

Uygulamayı çalıştırın ve **eğitmenler** sekmesini seçin. Sayfa ilgili OfficeAssignment varlıklarının Location özelliğini ve ilişkili OfficeAssignment varlığı olmadığında boş bir tablo hücresini görüntüler.

![Eğitmenler Dizin sayfası seçili öğe yok](read-related-data/_static/instructors-index-no-selection.png)

*Views/eğitmenler/Index. cshtml* dosyasında, Kapanış tablosu öğesinden sonra (dosyanın sonunda) aşağıdaki kodu ekleyin. Bu kod, bir eğitmen seçildiğinde bir eğitmenin ilgili kursların bir listesini görüntüler.

[!code-html[](intro/samples/cu/Views/Instructors/Index1.cshtml?range=66-101)]

Bu kod, `Courses` kurs listesini görüntülemek için görünüm modelinin özelliğini okur. Ayrıca, `Index` SEÇILI kursun kimliğini eylem yöntemine gönderen bir **seçme** Köprüsü de sağlar.

Sayfayı yenileyin ve bir eğitmen seçin. Artık Seçili eğitmenin atandığı kursları görüntüleyen bir kılavuz görürsünüz ve her kurs için atanan departmanın adını görürsünüz.

![Eğitmenler Dizin sayfası eğitmeni seçildi](read-related-data/_static/instructors-index-instructor-selected.png)

Yeni eklediğiniz kod bloğundan sonra aşağıdaki kodu ekleyin. Bu, kurs seçildiğinde bir kursa kaydedilen öğrencilerin listesini görüntüler.

[!code-html[](intro/samples/cu/Views/Instructors/Index1.cshtml?range=103-125)]

Bu kod, kursa kayıtlı öğrencilerin listesini görüntülemek için görünüm modelinin kayıtları özelliğini okur.

Sayfayı yeniden yenileyip bir eğitmen seçin. Ardından, kayıtlı öğrenciler ve bunların onların listesini görmek için bir kurs seçin.

![Eğitmenler Dizin sayfası eğitmeni ve kursu seçildi](read-related-data/_static/instructors-index.png)

## <a name="about-explicit-loading"></a>Açık yükleme hakkında

*InstructorsController.cs*' de eğitmenler listesini aldığınızda, `CourseAssignments` gezinti özelliği için bir Eager yüklemesi belirttiniz.

Kullanıcılardan yalnızca, seçili bir eğitmenin ve kursla kayıtlarını yalnızca nadiren görmek istediğini varsayalım. Bu durumda, kayıt verilerini yalnızca istenirse yüklemek isteyebilirsiniz. Açık yüklemenin nasıl yapılacağını gösteren bir örnek görmek için, `Index` yöntemini aşağıdaki kodla değiştirin, bu da kayıtları için Eager yüklemeyi kaldırır ve bu özelliği açıkça yükler. Kod değişiklikleri vurgulanır.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ExplicitLoading&highlight=23-29)]

Yeni kod, eğitmen varlıklarını alan koddan kayıt verileri için *Thenınclude* Yöntem çağrılarını bırakır. Ayrıca düşün `AsNoTracking`.  Bir eğitmen ve kurs seçilirse, vurgulanan kod seçili kurs için kayıt varlıklarını ve her kayıt için öğrenci varlıklarını alır.

Uygulamayı çalıştırın, şimdi eğitmenler dizin sayfasına gidin ve sayfada görüntülendikleriyle ilgili hiçbir fark görmezsiniz; ancak verilerin nasıl alındığını değiştirmiş olursunuz.

## <a name="get-the-code"></a>Kodu alma

[Tamamlanmış uygulamayı indirin veya görüntüleyin.](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="next-steps"></a>Sonraki adımlar

Bu öğreticide şunları yaptınız:

> [!div class="checklist"]
> * İlgili verileri yükleme hakkında öğrenilen
> * Bir kurslar sayfası oluşturuldu
> * Bir eğitmenler sayfası oluşturuldu
> * Açık yükleme hakkında bilgi edinildi

İlgili verileri güncelleştirme hakkında bilgi edinmek için sonraki öğreticiye ilerleyin.

> [!div class="nextstepaction"]
> [İlgili verileri güncelleştirme](update-related-data.md)
