Hi,

I've discovered another pattern, I'm not sure how to translate into a NgModule-less world.

The Nx community often uses the following structure:

```
   +-------> [FeatureLib | FeatureNgModule ] -------+
   |                                                V         
[App] ---> [FeatureLib | FeatureNgModule ] ---> [DataLib | DataNgModule]
                                                            ^
                                                            |

                                                    StoreModule.forFeature(...)
```

- DataLib: Takes care of state management for a given application area (=a bunch of features belonging together)
- For that, the DataLib registeres its own "feature store" (StoreModule.forFeature)
- FeatureLib: Implements one feature of this area on top of the DataLib

Now, without NgMoudles, this very structure might look like this:

```
   +---------> [FeatureLib] -----+
   |                             V
[App] ---> [FeatureLib] ---> [DataLib]
```

So, I'm wondering where to register the "feature store" (StoreModule.forFeature).


One Option that comes in mind: Put it into the FeatureLib's RouterConfig so it ends up in an enviroment injector.

```
   +-------> [FeatureLib | RouterConfig ] -------+
   |                                             V         
[App] ---> [FeatureLib | RouterConfig ] ---> [DataLib]
```

However, the Store (Slice) is reused among several Features (FeatureLibs) of the same application area.

To get the same behavior as before, we needed one (and only one) environment injector on top of all features of the same application area.

Idea: Add a dummy RouterConfig with a component-less route:


```
                         +----------> [FeatureLib | RouterConfig*1 ] ------------+
                         |                                                       V         
[App] ---> [AreaLib | RouterConfig] ---> [FeatureLib | RouterConfig*1 ] ---> [DataLib]
                         |                                                       ^
                         +-------------------------------------------------------+
                                            *2
```

*1 Perhaps we can move the FeatureLib's RouterConfig into the AreaLib too.
*2 Drawback: the AreaLib's Router config needs to be aware of the NGRX config in the DataLib. 

Are there any other approaches I'm missing here?
Is there another option of created an environment injector in the DataLib?

Using a router config in the data lib doesn't work, because this lib isn't routed ...

