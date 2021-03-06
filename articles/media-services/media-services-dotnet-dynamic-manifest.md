<properties 
	pageTitle="Criando filtros com o SDK do .NET dos Serviços de Mídia do Azure" 
	description="Este tópico descreve como criar filtros para que seu cliente possa usá-los na transmissão de seções específicas de um fluxo. Os Serviços de Mídia criam manifestos dinâmicos para atingir esse streaming seletivo." 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="ne" 
	ms.topic="article" 
	ms.date="04/18/2016"
	ms.author="juliako;cenkdin"/>


#Criando filtros com o SDK do .NET dos Serviços de Mídia do Azure

> [AZURE.SELECTOR]
- [.NET](media-services-dotnet-dynamic-manifest.md)
- [REST](media-services-rest-dynamic-manifest.md)

A partir da versão 2.11, os Serviços de Mídia permitem definir filtros para seus ativos. Esses filtros são regras do lado do servidor que permitirão aos clientes optar por realizar ações como: reproduzir apenas uma seção de um vídeo (em vez de reproduzir o vídeo inteiro) ou especificar apenas um subconjunto de representações de áudio e vídeo com o qual o dispositivo do cliente pode lidar (em vez de todas as representações que estão associadas ao ativo). A filtragem de ativos é obtida por meio de **Manifestos Dinâmicos** criados mediante solicitação do cliente para transmitir um vídeo com base em filtros especificados.

Para obter mais informações relacionadas a filtros e ao Manifesto Dinâmico, consulte [Visão geral de manifestos dinâmicos](media-services-dynamic-manifest-overview.md).

Este tópico mostra como usar .NET SDK dos Serviços de Mídia para criar, atualizar e excluir filtros.


Observe que se atualiza um filtro, talvez sejam necessários até 2 minutos para que o ponto de extremidade do streaming atualize as regras. Se o conteúdo foi servido usando esse filtro (e armazenado em cache nos proxies e caches CDN), atualizar esse filtro pode resultar em falhas do player. É recomendável limpar o cache depois de atualizar o filtro. Se essa opção não for possível, considere usar um filtro diferente.

##Tipos usados para criar filtros

Os tipos a seguir são usados durante a criação de filtros:

- **IStreamingFilter**. Esse tipo é baseado na seguinte API REST [filtro](http://msdn.microsoft.com/library/azure/mt149056.aspx)
- **IStreamingAssetFilter**. Esse tipo é baseado na seguinte API REST [AssetFilter](http://msdn.microsoft.com/library/azure/mt149053.aspx)
- **PresentationTimeRange** Esse tipo é baseado na seguinte API REST [PresentationTimeRange](http://msdn.microsoft.com/library/azure/mt149052.aspx)
- **FilterTrackSelectStatement** e **IFilterTrackPropertyCondition**. Esses tipos são baseados nas seguintes APIs REST [FilterTrackSelect e FilterTrackPropertyCondition](http://msdn.microsoft.com/library/azure/mt149055.aspx)


##Criar/atualizar/ler/excluir os filtros globais

O código a seguir mostra como usar o .NET para criar, atualizar, ler e excluir filtros de ativo.
	
	string filterName = "GlobalFilter_" + Guid.NewGuid().ToString();
	            
	List<FilterTrackSelectStatement> filterTrackSelectStatements = new List<FilterTrackSelectStatement>();
	
	FilterTrackSelectStatement filterTrackSelectStatement = new FilterTrackSelectStatement();
	filterTrackSelectStatement.PropertyConditions = new List<IFilterTrackPropertyCondition>();
	filterTrackSelectStatement.PropertyConditions.Add(new FilterTrackNameCondition("Track Name", FilterTrackCompareOperator.NotEqual));
	filterTrackSelectStatement.PropertyConditions.Add(new FilterTrackBitrateRangeCondition(new FilterTrackBitrateRange(0, 1), FilterTrackCompareOperator.NotEqual));
	filterTrackSelectStatement.PropertyConditions.Add(new FilterTrackTypeCondition(FilterTrackType.Audio, FilterTrackCompareOperator.NotEqual));
	filterTrackSelectStatements.Add(filterTrackSelectStatement);
	
	// Create
	IStreamingFilter filter = _context.Filters.Create(filterName, new PresentationTimeRange(), filterTrackSelectStatements);
	
	// Update
	filter.PresentationTimeRange = new PresentationTimeRange(timescale: 500);
	filter.Update();
	
	// Read
	var filterUpdated = _context.Filters.FirstOrDefault();
	Console.WriteLine(filterUpdated.Name);

	// Delete
	filter.Delete();


##Criar/atualizar/ler/excluir os filtros de ativo

O código a seguir mostra como usar o .NET para criar, atualizar, ler e excluir filtros de ativo.

	
	string assetName = "AssetFilter_" + Guid.NewGuid().ToString();
	var asset = _context.Assets.Create(assetName, AssetCreationOptions.None);
	
	string filterName = "AssetFilter_" + Guid.NewGuid().ToString();
	
	    
	// Create
	IStreamingAssetFilter filter = asset.AssetFilters.Create(filterName,
	                                    new PresentationTimeRange(), 
	                                    new List<FilterTrackSelectStatement>());
	
	// Update
	filter.PresentationTimeRange = 
	        new PresentationTimeRange(start: 6000000000, end: 72000000000);
	
	filter.Update();
	
	// Read
	asset = _context.Assets.Where(c => c.Id == asset.Id).FirstOrDefault();
	var filterUpdated = asset.AssetFilters.FirstOrDefault();
	Console.WriteLine(filterUpdated.Name);
	
	// Delete
	filterUpdated.Delete();
	



##Criar URLs de streaming que usam filtros

Para obter informações sobre como publicar e distribuir seus ativos, consulte [Visão geral do fornecimento de conteúdo a clientes](media-services-deliver-content-overview.md).


Os exemplos a seguir mostram como adicionar filtros às URLs de streaming.

**MPEG DASH**

	http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=mpd-time-csf, filter=MyFilter)

**Apple HTTP Live Streaming (HLS) V4**

	http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl, filter=MyFilter)

**Apple HTTP Live Streaming (HLS) V3**

	http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl-v3, filter=MyFilter)

**Streaming Suave**

	http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(filter=MyFilter)


**HDS**

	http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=f4m-f4f, filter=MyFilter)


##Roteiros de aprendizagem dos Serviços de Mídia

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##Fornecer comentários

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]


##Consulte também 

[Visão geral de manifestos dinâmicos](media-services-dynamic-manifest-overview.md)
 

<!---HONumber=AcomDC_0420_2016-->