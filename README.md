OOOR - OpenObject On Rails
====

<table>
    <tr>
        <td><a href="http://github.com/rvalyi/ooor" title="OOOR - OpenObject On Rails"><img src="http://sites.google.com/site/assetserversite/_/rsrc/1257126460164/home/ooor_s.jpg" width="159px" height="124px" /></a></td>
        <td><b>BY</b></td>
        <td><a href="http://www.akretion.com" title="Akretion - open source to spin the world"><img src="http://sites.google.com/site/assetserversite/_/rsrc/1257126470309/home/akretion_s.png" width="228px" height="124px" /></a></td>
        <td>
OOOR stands for OpenObject On Rails. OpenObject is the RAD framework behind OpenERP,
the ERP that doesn't hurt, just like Rails is "web development that doesn't hurt".
So OOOR exposes seamlessly your OpenOpbject application, to your custom Rails application.
Needless to say, OOOR doubly doesn't hurt.
Furthermore, OOOR only depends on the "activeresource" gem. So it can even be used
in any (J)Ruby application without Rails.
        </td>
    </tr>
</table>


Why?
------------

OpenERP makes it really straightforward to create/customize business applications with:

* standard ERP business modules (more than 300 modules)
* complex relationnal data model, with automated migration and backoffice interfaces
* ACID transactions on PostgreSQL
* role based
* modular
* integrated BPM (Business Process Management)
* integrated reporting system, with integrated translations
* both native GTK/QT clients and standard web access

In a word OpenERP really shines when it's about quickly creating the backoffice of those enterprise applications.
OpenERP is a bit higher level than Rails (for instance it's component oriented while Rails is REST oriented) so if you adhere to the OpenERP conventions, 
then you are done faster than coding a Rails app (seriously).
Adhering means: you stick to OpenObject views, widgets, look and feel, components composition, ORM (kind of ActiveRecord), the Postgres database...

But sometimes you can't afford that. Typicall examples are B2C end users applications like e-commerce shops.
So what happens if you don't adhere to the OpenERP framework?
Well that's where OOOR comes into action. It allows you to build a Rails application much like you want, where you totally control the end user presentation and interaction.
But OOOR makes it straightforward to use a standard OpenERP models as your persistent models.

An other reason why you might want to use OOOR is because you would like to code essentially a Rails or say web application
(because you know it better, because the framework is cleaner or because you will reuse something, possibly Java libraries though JRuby)
but you still want to benefit from OpenERP features.

Yet an other typicall use case would be to test your OpenERP application/module using Rails best of bread BDD Ruby frameworks such as RSpec or Cucumber.

Finally you might also want to use OOOR simply to expose your OpenERP through REST to other consumer applications. Since OOOR just does that too out of the box.



How?
------------

OpenERP is a Python based open source ERP. Every action in OpenERP is actually invokable as a webservice (SOA orientation, close to being RESTful).
OOOR just takes advantage of brings this power your favorite web development tool - Rails - with OpenERP domain objects and business methods.

OOOR aims at being a very simple piece of code (< 500 lines of code; e.g no bug, easy to evolve) adhering to Rails standards.
So instead of re-inventing the wheel, OOOR basically just sits on the top of Rails [ActiveResource::Base](http://api.rubyonrails.org/classes/ActiveResource/Base.html), the standard way of remoting you ActiveRecord Rails models with REST.

Remember, ActiveResource is actually simpler than [ActiveRecord](http://api.rubyonrails.org/classes/ActiveRecord/Base.html). It's aimed at remoting ANY object model, not necessarily ActiveRecord models.
So ActiveResource is only a subset of ActiveRecord, sharing the common denominator API (integration is expected to become even more powerful in Rails 3).

OOOR implements ActiveResource public API almost fully. It means that you can remotely work on any OpenERP model using [the standard ActiveResource API](http://api.rubyonrails.org/classes/ActiveResource/Base.html).

But, OOOR goes actually a bit further: it does implements model associations (one2many, many2many, many2one, single table inheritance).
Indeed, when loading the OpenERP models, we load the relational meta-model using OpenERP standard datamodel introspection services.
Then we cache that relational model and use it in OpenObjectResource.method_missing to load associations as requested.

OOOR also extends ActiveResource a bit with special request parameters (like :domain or :context) that will just map smoothly to the OpenERP native API, see API.


Installation
------------

You can use OOOR in a standalone (J)Ruby application, or in a Rails application.
For both example we assume that you already started some OpenERP server on localhost, with XML/RPC on port 8069 (default),
with a database called 'mybase', with username 'admin' and password 'admin'.

In all case, you first need to install the ooor gem:

    $ gem install ooor
(the ooor gem is hosted [on gemcutter.org here](http://gemcutter.org/gems/ooor), make sure you have it in your gem source lists, a way is to do >gem tumble)


### Standalone (J)Ruby application:

Let's test OOOR in an irb console (irb command):
    $ require 'rubygems'
    $ require 'ooor'
    $ include Ooor
    $ Ooor.reload!({:url => 'http://localhost:8069/xmlrpc', :database => 'mybase', :username => 'admin', :password => 'admin'})
This should load all your OpenERP models into Ruby proxy Activeresource objects. Of course there are option to load only some models.
Let's try to retrieve the user with id 1:
    $ ResUsers.find(1)
    
    
### (J)Ruby on Rails application:

we assume you created a working Rails application, in your config/environment.rb
Inside the Rails::Initializer.run do |config| statement, paste the following gem dependency:

    $ config.gem "ooor"

Now, you should also create a ooor.yml config file in your config directory
You can copy/paste [the default ooor.yml from the OOOR gem](http://github.com/rvalyi/ooor/blob/master/ooor.yml)
and then adapt it to your OpenERP server environment.
If you set the 'bootstrap' parameter to true, OpenERP models will be loaded at the Rails startup.
That the easiest option to get started while you might not want that in production.

Then just start your Rails application, your OpenERP models will be loaded as you'll see in the Rails log.
You can then use all the OOOR API upon all loaded OpenERP models in your regular Rails code (see API usage section).
A good way to start playing with OOOR is inside the console, using:
    $ ruby script/console #or jruby script/console on JRuby of course

Enabling REST HTTP routes to your OpenERP models:
in your config/route.rb, you can alternatively enable routes to all your OpenERP models by addding:
    $ OpenObjectsController.load_all_controllers(map)

Or only enable the route to some specific model instead (here partners):
    $ map.resources :res_partner



API usage
------------

Note: Ruby proxies objects are named after OpenERP models in but removing the '.' and using CamelCase.
we remind you that OpenERP tables are also named after OpenERP models but replacing the '.' by '_'.

Basic finders:

    $ ProductProduct.find(1)
    $ ProductProduct.find([1,2])
    $ ProductProduct.find([1])
    $ ProductProduct.find(:all)
    $ ProductProduct.find(:last)


OpenERP domain support:

    $ ResPartner.find(:all, :domain=>[['supplier', '=', 1],['active','=',1]])
More subbtle now, remember OpenERP use a kind of inverse polish notation for complex domains,
here we look for a product in category 1 AND which name is either 'PC1' OR 'PC2':
    $ ProductProduct.find(:all, :domain=>[['categ_id','=',1],'|',['name', '=', 'PC1'],['name','=','PC2']])


OpenERP context support:

    $ ProductProduct.find(1, :context => {:my_key => 'value'})


Request params or ActiveResource equivalence of OpenERP domain (but degraded as only the = operator is supported, else use domain):

    $ Partners.find(:all, :params => {:supplier => true})


OpenERP search method:

    $ ResPartner.search([['name', 'ilike', 'a']], 0, 2)
Arguments are: domain, offset=0, limit=false, order=false, context={}, count=false


Relations (many2one, one2many, many2many) support:

    $ SaleOrder.find(1).order_line
    $ p = ProductProduct.find(1)
    $ p.product_tmpl_id #many2one relation
    $ p.product_tmpl_id.taxes_id = [6, 0, [1,2]] #create many2many associations,
    $ p.save #assigns taxes with id 1 and 2 as sale taxes,
see [the official OpenERP documentation](http://doc.openerp.com/developer/5_18_upgrading_server/19_1_upgrading_server.html?highlight=many2many)


Inherited relations support:

    $ ProductProduct.find(1).categ_id #where categ_id is inherited from the ProductTemplate

Please notice that loaded relations are cached (to avoid  hitting OpenERP over and over)
until the root object is reloaded (after save/update for instance).
Currently, save/update doesn't save the whole object graph but only the current object.
We might change this in the future to match the way OpenERP clients are working which
is supported by the OpenERP ORM, see issue: http://github.com/rvalyi/ooor/issues/#issue/3


Load only specific fields support (faster than loading all fields):

    $ ProductProduct.find(1, :fields=>["state", "id"])
    $ ProductProduct.find(:all, :fields=>["state", "id"])
    $ ProductProduct.find([1,2], :fields=>["state", "id"])
    $ ProductProduct.find(:all, :fields=>["state", "id"])
    even in relations:
    $ SaleOrder.find(1).order_line(:fields => ["state"])


Create:

    $ pc = ProductCategory.new(:name => 'Categ From Rails!')
    $ #<ProductCategory:0xb702c42c @prefix_options={}, @attributes={"name"=>"Categ From Rails!"}>
    $ pc.create
    $ => 14


Update:

    $ pc.name = "A new name"
    $ pc.save


Copy:

    $ copied_object = pc.copy({:categ_id => 2})  #first optionnal arg is new default values, second is context


Delete:

    $ pc.destroy


Call workflow:

    $ s = SaleOrder.find(2)
    $ s.wkf_action('cancel')
    $ s.state
    $ => 'cancel'


On Change methods:

    $ l=SaleOrderLine.new
    $ l.on_change('product_id_change', 1, 1, false, false, false, false, false, false, 1)
    $ => #<Ooor::SaleOrderLine:0x10c2cfe1 @prefix_options={}, @attributes={"product_packaging"=>false, "product_uos_qty"=>false, "th_weight"=>0}>
Notice that it reloads the Objects attrs and print warning message accordingly


Call aribtrary method:

    $ use static ObjectClass.rpc_execute_with_all method
    $ or object.call(method_name, args*) #were args is an aribtrary list of arguments
    $ or use the method missing wrapper that will proxy any OpenERP osv.py/orm.py method, see fo instance:
    $ ResPartner.name_search('ax', [], 'ilike', {})
    $ ProductProduct.fields_view_get(132, 'tree', {})


Call old wizards:

    $ inv = AccountInvoice.find(4)
    $ wizard = inv.old_wizard_step('account.invoice.pay') #tip: you can inspect the wizard fields, arch and datas
    $ wizard.reconcile({:journal_id => 6, :name =>"from_rails"}) #if you want to pay all; will give you a reloaded invoice
    $ #or if you want a payment with a write off:
    $ wizard.writeoff_check({"amount" => 12, "journal_id" => 6, "name" =>'from_rails'}) #use the button name as the wizard method
    $ wizard.reconcile({required missing write off fields...}) #will give you a reloaded invoice because state is 'end'
    $ TODO test and document new osv_memory wizards API


Change logged user:

    $ Ooor.global_login('demo', 'demo')
    $ s = SaleOrder.find(2)
    $ => 'Access denied error'
Notice that this changes the globally logged user and doesn't address the issue of
different users using Ooor concurrently with different credentials as you might want
for instance in a Rails web application. That second issue will be addressed at
the API level but it's not easy as ActiveResource has not been designed for it.


Change log level:

By default the log level is very verbose (debug level) to help newcomers to jumpstart.
However you might want to change that. 2 solutions:
    $ Ooor.logger.level = 1 #available levels are those of the standard Ruby Logger class: 0 debug, 1 info, 2 error
    $ In the config yaml file or hash, set the :log_level parameter



REST HTTP API:

    $ http://localhost:3000/res_partner
    $ http://localhost:3000/res_partner.xml
    $ http://localhost:3000/res_partner.json
    $ http://localhost:3000/res_partner/2
    $ http://localhost:3000/res_partner/2.json
    $ http://localhost:3000/res_partner/2.xml
    $ http://localhost:3000/res_partner/[2,3,4].xml
    $ http://localhost:3000/res_partner/[2,3,4].json

    $ TODO http://localhost:3000/res_partner.xml?active=1

    $ TODO http://localhost:3000/res_partner.xml?domain=TODO


FAQ
------------

### How do I know which object or OpenERP webservice I should Invoke from OOOR?

An easy is to use your GTK client and start it with the -l debug_rpc (or alternatively -l debug_rpc_answer) option.
For non *nix users, you can alternatively start your server with the --log-level=debug_rpc option (you can also set this option in your hidden OpenERP server config file in your user directory).
Then create indents in the log before doing some action and watch your logs carefully. OOOR will allow you to do the same easily from Ruby/Rails.

### How can I load/reload my OpenERP models into my Ruby application?

You can load/reload your models at any time (even in console), using the Ooor.reload! method, for instance:
    $ Ooor.reload!({:url => 'http://localhost:8069/xmlrpc', :database => 'mybase', :username => 'admin', :password => 'admin'})
or using a config YAML file instead:
    $ Ooor.reload!("config/ooor.yml")

### Do I need to load all the OpenERP models in my Ruby application?

You can load only some OpenERP models (not all), which is faster and better in term of memory/security:
    $ Ooor.reload!({:models => [res.partner, product.template, product.product], :url => 'http://localhost:8069/xmlrpc', :database => 'mybase', :username => 'admin', :password => 'admin'})


### Isn't OOOR slow?

You might think that proxying a Python based server through a Rails app using XML/RPC would be dog slow. Well, not too much actually.
I did some load testing using a Centrino Duo laptop (dual core at 1.6 GHz), and I was able to approach 10 requests/sec, to display some OpenERP resource as XML through HTTP.
That should be enough for what it's meant too. Heavily loaded application might cache things at the Rails level if that's too slow, but I don't think so.

Also notice that using JRuby I could serve some 20% faster.

### JRuby compatibility

Yes Ooor is fully JRuby compatible. It's even somewhat 20% faster using Java6 + last JRuby.
This might be espcially interresting if you plan to mix Java libraries with OpenERP in the same web appication.

### Can I extend the OOOR models?

Yes you can perfectly do that. Basically an OpenERP model get a basic ActiveResource model. For instance: product.product is mapped to the ProductProduct ActiveResource model.
But using the Ruby open classes features, you can asbolutely re-open the ProductProduct class and add features of your own.
In you app/model directory, create a product_product.rb file with inside, the redéfinition of ProductProduct, for instance:

    $ class ProductProduct < OpenObjectResource
    $   def foo
    $     "bar"
    $   end
    $ end

Now a ProductProduct resource got a method foo, returning "bar".

### Can I extend the OOOR controllers?

Yes, you can do that, see the "how to extends models" section as it's very similar. Instead, in the app/controllers directory, you'll re-open defined controllers,
for the product.product controllers, it means creating a product_product_controller.rb file with:

    $ class ProductProduct < OpenObjectsController
    $   def foo
    $     render :text => "bar"
    $   end
    $ end

Now, if you register that new method in your route.rb file GET /product_product/1/foo will render "bar" on your browser screen.
You could instead just customize the existing CRUD methods so you don't need to regiter any other route in route.rb.