# Capítol 4: Seguretat - Una breu introducció

En el [capítol anterior](../03_modelbasic/), vam crear la nostra primera taula destinada a emmagatzemar dades de negoci. En una aplicació de negoci com Odoo, una de les primeres qüestions a tindre en compte és qui pot accedir a les dades. Odoo proporciona un mecanisme de seguretat per a permetre l'accés a les dades a grups específics d'usuaris.

Aquest capítol té com a objectiu cobrir el mínim necessari per al nostre nou mòdul.

## Fitxers de dades (CSV)

Odoo és un sistema altament basat en dades. Encara que el comportament es personalitza utilitzant codi Python, part del valor d'un mòdul resideix en les dades que configura quan es carrega. Una manera de carregar dades és mitjançant un fitxer CSV. Un exemple és la [llista d'estats de països]({GITHUB_PATH}/odoo/addons/base/data/res.country.state.csv) que es carrega en instal·lar el mòdul `base`.

```text
"id","country_id:id","name","code"
state_au_1,au,"Australian Capital Territory","ACT"
state_au_2,au,"New South Wales","NSW"
state_au_3,au,"Northern Territory","NT"
state_au_4,au,"Queensland","QLD"
...
```

- `id` és un *identificador extern* (*external identifier*). Es pot utilitzar per a fer referència al registre (sense conéixer el seu identificador a la base de dades).
- `country_id:id` fa referència al país utilitzant el seu *identificador extern*.
- `name` és el nom de l'estat.
- `code` és el codi de l'estat.

Aquests tres camps estan [definits](https://github.com/odoo/odoo/blob/2ad2f3d6567b6266fc42c6d2999d11f3066b282c/odoo/addons/base/models/res_country.py#L108-L111) en el model `res.country.state`.

Per convenció, un fitxer que importa dades se situa a la carpeta `data` d'un mòdul. Quan les dades estan relacionades amb la seguretat, se situen a la carpeta `security`. Quan les dades estan relacionades amb vistes i accions (això ho tractarem més avant), se situen a la carpeta `views`. A més, tots aquests fitxers s'han de declarar a la llista `data` dins del fitxer `__manifest__.py`. El nostre fitxer d'exemple està definit [al manifest del mòdul base](https://github.com/odoo/odoo/blob/e8697f609372cd61b045c4ee2c7f0fcfb496f58a/odoo/addons/base/__manifest__.py#L29).

Tingueu en compte també que el contingut dels fitxers de dades només es carrega quan un mòdul s'instal·la o s'actualitza.

!!! warning "Compte amb l'ordre!"
    Els fitxers de dades es carreguen seqüencialment seguint el seu ordre al fitxer `__manifest__.py`. Això significa que si la dada `A` fa referència a la dada `B`, heu d'assegurar-vos que `B` es carregue abans que `A`.

    En el cas dels estats dels països, notareu que la [llista de països](https://github.com/odoo/odoo/blob/e8697f609372cd61b045c4ee2c7f0fcfb496f58a/odoo/addons/base/__manifest__.py#L22) es carrega **abans** que la [llista d'estats de països](https://github.com/odoo/odoo/blob/e8697f609372cd61b045c4ee2c7f0fcfb496f58a/odoo/addons/base/__manifest__.py#L29). Això és perquè els estats fan referència als països.

Per què és important tot això per a la seguretat? Perquè tota la configuració de seguretat d'un model es carrega a través de fitxers de dades, com veurem a la secció següent.

## Drets d'accés

!!! info "Referència"
    La documentació relacionada amb aquest tema es pot trobar als [drets d'accés (ACL)](reference/security/acl).

!!! abstract "Objectiu"
    Al final d'aquesta secció, l'avís següent ja no hauria d'aparéixer:

    ```text
    WARNING rd-demo odoo.modules.loading: The models ['estate.property'] have no access rules...
    ```

Quan no es defineixen drets d'accés en un model, Odoo determina que cap usuari pot accedir a les dades. Fins i tot es notifica al registre (log):

```text
WARNING rd-demo odoo.modules.loading: The models ['estate.property'] have no access rules in module estate, consider adding some, like:
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
```

Els drets d'accés es defineixen com a registres del model `ir.model.access`. Cada dret d'accés s'associa a un model, a un grup (o a cap grup per a un accés global) i a un conjunt de permisos: crear, llegir, escriure i desvincular (unlink)[^unlink]. Aquests drets d'accés se solen definir en un fitxer CSV anomenat `ir.model.access.csv`.

Ací teniu un exemple per al nostre `test_model` anterior:

```text
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_test_model,access_test_model,model_test_model,base.group_user,1,0,0,0
```

- `id` és un *identificador extern*.
- `name` és el nom de l'`ir.model.access`.
- `model_id/id` fa referència al model al qual s'aplica el dret d'accés. La forma estàndard de fer referència al model és `model_<model_name>`, on `<model_name>` és el `_name` del model amb el `.` substituït per `_`. Vos pareix feixuc? Certament ho és...
- `group_id/id` fa referència al grup al qual s'aplica el dret d'accés.
- `perm_read,perm_write,perm_create,perm_unlink`: permisos de lectura, escriptura, creació i desvinculació (unlink).

!!! example "Exercici: afegiu drets d'accés"
    Creeu el fitxer `ir.model.access.csv` a la carpeta adequada i definiu-lo al fitxer `__manifest__.py`.

    Doneu els permisos de lectura, escriptura, creació i desvinculació al grup `base.group_user`.

    Consell: el missatge d'avís del registre (log) vos dona la major part de la solució ;-)

Reinicieu el servidor i el missatge d'avís hauria d'haver desaparegut!

Ara és el moment de, finalment, [interactuar amb la interfície d'usuari](05_firstui)!