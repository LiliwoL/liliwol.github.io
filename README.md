---
description: Créer un module Ajouter une étiquette de temps pour Magento 2
---

# Premier module

Dans ce module, vous apprendrez à créer un module personnalisé pour ajouter et afficher l'étiquette de temps sur la page du produit unique, l'ajouter à la page du panier, à la page d'affichage et au panneau d'administration.

Fondamentalement, ce module est défini en fonction de l'état du stock en fonction des produits individuels.

En commençant par le module Structure des dossiers :

![](https://cynoteck.com/wp-content/uploads/2021/09/image001.png)

Pour créer un module personnalisé, vous devrez effectuer les étapes suivantes&#x20;

#### **Étape 1 : Créez un dossier avec le nom LeadTime module**

Le nom du module est défini comme "**Nom du fournisseur\_Nom du module**”. La première partie est le nom du fournisseur, et la dernière partie est le nom du module :

Par exemple – Mon nom de module est **LeadTime\_Label**, concentrez-vous sur le guide suivant pour créer les dossiers :

Les emplacements des dossiers pour les modules dans Magento 2 : **dossier app/code**&#x20;

`app/code/LeadTime/Libellé`

#### **Étape 2 : Création d'un fichier registration.php**

Créer un **registration.php** fichier qui enregistre le module avec le nom du dossier et du sous-dossier du modèle.

**app/code/LeadTime/registration.php**

****

```
<?php
\Magento\Framework\Component\ComponentRegistrar::register(
\Magento\Framework\Component\ComponentRegistrar::MODULE,
'LeadTime_Label',
__DIR__
);
```

#### **Étape 3 : Création du fichier etc/module.xml**

Il est nécessaire de créer un dossier etc et d'ajouter le **module.xml** filet

**app/code/LeadTime/Label/etc/module.xml**

```
<?xml version="1.0"?>

<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
<module name="LeadTime_Label" setup_version="1.0.0">
</module>
</config>
```

#### **Étape 4 : Création du fichier etc/events.xml**

Les événements sont exécutés par le module lorsque certaines actions sont déclenchées. Magento nous permet de créer nos propres événements qui peuvent être envoyés en code. Lorsqu'un événement est distribué, il peut transmettre des données à n'importe quel observateur configuré pour surveiller cet événement.

Dans ce fichier d'événements, nous déclenchons deux événements ; un événement est appelé après l'ajout du produit au panier de paiement, le module appelle **CaissePanierAjouterObservateur** Observer, et le deuxième événement avant devis soumettre qui appelle **CitationEnvoyerObservateur** Observateur.

**app/code/LeadTime/Label/etc/event.xml**

****

```
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Event/etc/events.xsd">
<event name="checkout_cart_product_add_after">
<observer name="extension_checkout_cart_product_add_after" instance="LeadTime\Label\Observer\CheckoutCartAddObserver" />
</event>
<event name="sales_model_service_quote_submit_before">
<observer name="unique_name" instance="LeadTime\Label\Observer\QuoteSubmitObserver" />
</event>
</config>
```

#### **Étape 5 : Création du fichier Observer/CheckoutCartAddObserver.php**

Ces observateurs sont une classe Magento qui peut affecter les performances de toute logique métier. L'Observer est exécuté lorsque les événements configurés pour les surveiller sont distribués par le gestionnaire d'événements.

Dans un premier temps, nous allons créer un dossier nommé **Observateur** à la racine du module.

Après cela, nous allons créer un fichier PHP pour le premier événement auquel nous avons donné le nom de l'observateur dans le fichier d'événement.

Dans le **CaissePanierAjouterObservateur** fichier, nous vérifions la quantité de produit si la quantité est supérieure à zéro, puis définissons l'étiquette '**Délai de livraison du produit : 7 -21 jours**', si la quantité est nulle ou inférieure à zéro, définissez l'étiquette '**Délai de livraison des commandes spéciales : 16 - 22 semaines**»

**app/code/LeadTime/Label/Observer/CheckoutCartAddObserver.php**

****

```
<?php
namespace LeadTime\Label\Observer;

use Magento\Framework\Event\Observer as EventObserver;
use Magento\Framework\Event\ObserverInterface;
use Magento\Store\Model\StoreManagerInterface;
use Magento\Framework\View\LayoutInterface;
use Magento\Framework\App\RequestInterface;
use Magento\Framework\Serialize\SerializerInterface;

class CheckoutCartAddObserver implements ObserverInterface
{
protected $request;
private $serializer;
protected $layout;
protected $storeManager;
public function __construct(RequestInterface $request, SerializerInterface $serializer, StoreManagerInterface $storeManager, LayoutInterface $layout)
{
$this->_request = $request;
$this->serializer = $serializer;
$this->layout = $layout;
$this->storeManager = $storeManager;
}
public function execute(\Magento\Framework\Event\Observer $observer){
$item = $observer->getQuoteItem();
$additionalOptions = array();
$product = $observer->getProduct();
$productId=$product->getId();
$objectManager = \Magento\Framework\App\ObjectManager::getInstance();
$StockState = $objectManager->get('\Magento\CatalogInventory\Api\StockStateInterface');
$qty=$StockState->getStockQty($product->getId(), $product->getStore()->getWebsiteId());
if($qty > 0){
$label='Product Instock Lead Time: 7 -21 days' ;
}else{
$label='Special Order Lead Time: 16 - 22 Weeks' ;
}
if ($additionalOption = $item->getOptionByCode('additional_options')) {
$additionalOptions = $this->serializer->unserialize($additionalOption->getValue());
}
$additionalOptions[] = [
'label' => 'Lead Time',
'value' => $label
];
if (!is_null($additionalOptions)) {
$item->addOption(array(
'product_id' => $item->getProductId(),
'code' => 'additional_options',
'value' => $this->serializer->serialize($additionalOptions)
));
}
}
}
```

#### **Étape 6 : Création du fichier Observer/QuoteSubmitObserver.php**

Dans ce fichier, nous hériterons de la classe ObserverInterface. Et obtiendra le devis du produit, la valeur de la commande et le définira dans le choix d'une option supplémentaire.

**app/code/LeadTime/Label/Observer/QuoteSubmitObserver.php**

****

```
<?php
namespace LeadTime\Label\Observer;
use Magento\Framework\Event\ObserverInterface;
use Magento\Framework\Serialize\SerializerInterface;
class QuoteSubmitObserver implements ObserverInterface
{
private $serializer;
public function __construct(SerializerInterface $serializer)
{
$this->serializer = $serializer;
}
public function execute(\Magento\Framework\Event\Observer $observer)
{
try {
$quote = $observer->getQuote();
$order = $observer->getOrder();
$quoteItems = [];

// Map Quote Item with Quote Item Id
foreach ($quote->getAllItems() as $quoteItem) {
$quoteItems[$quoteItem->getId()] = $quoteItem;
}

foreach ($order->getAllVisibleItems() as $orderItem) {
$quoteItemId = $orderItem->getQuoteItemId();
$quoteItem = $quoteItems[$quoteItemId];
$additionalOptions = $quoteItem->getOptionByCode('additional_options');

if (!is_null($additionalOptions)) {
// Get Order Item's other options
$options = $orderItem->getProductOptions();
// Set additional options to Order Item
$options['additional_options'] = $this->serializer->unserialize($additionalOptions->getValue());
$orderItem->setProductOptions($options);
}
}
} catch (\Exception $e) {
// catch error if any
}
}
}
```

#### **Étape 7 : Création du fichier etc/di.xml**

Nous créons le **di.xml** fichier dans le dossier etc. Configure les dépendances injectées par le gestionnaire d'objets. Spécifiez également des paramètres de configuration sensibles à l'aide de ce fichier. Nous configurons notre plugin personnalisé pour définir des étiquettes de commande dans ce fichier particulier sur le panneau d'administration.

#### **Étape 8 : Créez le fichier Plugin/SetOrderItemValue.php**

On crée un dossier dans une racine de module avec un nom de plugin, puis à l'intérieur, on crée un fichier Php nommé **SetOrderItemValue.php** qui fixe notre étiquette à notre commande.



```
<?php
namespace LeadTime\Label\Plugin;
use Magento\Framework\Serialize\SerializerInterface;
class SetOrderItemValue
{
private $serializer;
public function __construct(SerializerInterface $serializer)
{
$this->serializer = $serializer;
}
public function aroundConvert(\Magento\Quote\Model\Quote\Item\ToOrderItem $subject, callable $proceed, $quoteItem, $data)
{

// get order item
$orderItem = $proceed($quoteItem, $data);

if(!$orderItem->getParentItemId() && $orderItem->getProductType() == \Magento\Catalog\Model\Product\Type::TYPE_SIMPLE){
if ($additionalOptionsQuote = $quoteItem->getOptionByCode('additional_options')) {
//To do
// - check to make sure element are not added twice
// - $additionalOptionsQuote - may not be an array
if($additionalOptionsOrder = $orderItem->getProductOptionByCode('additional_options')){
$additionalOptions = array_merge($additionalOptionsQuote, $additionalOptionsOrder);
}
else{
$additionalOptions = $additionalOptionsQuote;
}
if(!is_null($additionalOptions)){
$options = $orderItem->getProductOptions();
$options['additional_options'] = $this->serializer->unserialize($additionalOptions->getValue());
$orderItem->setProductOptions($options);
}
}
}

return $orderItem;
}
}
```

#### **Étape 9 : Activation du module**

Après avoir créé le module, exécutez la commande suivante :

&#x20;module php bin/magento:état

affiche la liste des modules désactivés : **LeadTime\_Label** mon module.

![](https://cynoteck.com/wp-content/uploads/2021/09/image003.png)

activez le module maintenant, exécutez la commande en tant que :

**module php bin/magento : activer LeadTime\_Label**

![](https://cynoteck.com/wp-content/uploads/2021/09/image006.jpg)

#### **Étape 10 : Après avoir activé le module Exécutez les commandes Magento**

php bin / magento setup: mise à jour

configuration de php bin / magento: static-content: deploy -f

php bin / magento c: c

php bin/magento c:f

chmod -R 777 généré/ var/ pub/

Page du produit du site Web, page du panier d'annonces et page de paiement après l'installation :

![](https://cynoteck.com/wp-content/uploads/2021/09/image008.jpg)

![](https://cynoteck.com/wp-content/uploads/2021/09/image009.png)

![](https://cynoteck.com/wp-content/uploads/2021/09/image011.png)

![](https://cynoteck.com/wp-content/uploads/2021/09/image013.png)
