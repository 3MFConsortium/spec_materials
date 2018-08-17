# ![3mf logo](images/3mf_logo_50px.png) 3D Manufacturing Format

# Materials and Properties Extension

| **Version** | 1.2 |
| --- | --- |
| **Status** | Draft |


## Disclaimer

THESE MATERIALS ARE PROVIDED "AS IS." The contributors expressly disclaim any warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to the materials. The entire risk as to implementing or otherwise using the materials is assumed by the implementer and user. IN NO EVENT WILL ANY MEMBER BE LIABLE TO ANY OTHER PARTY FOR LOST PROFITS OR ANY FORM OF INDIRECT, SPECIAL, INCIDENTAL, OR CONSEQUENTIAL DAMAGES OF ANY CHARACTER FROM ANY CAUSES OF ACTION OF ANY KIND WITH RESPECT TO THIS DELIVERABLE OR ITS GOVERNING AGREEMENT, WHETHER BASED ON BREACH OF CONTRACT, TORT (INCLUDING NEGLIGENCE), OR OTHERWISE, AND WHETHER OR NOT THE OTHER MEMBER HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.


## Table of Contents



## Preface

### About this Specification
This 3MF materials and properties specification is an extension to the core 3MF specification. This document cannot stand alone and only applies as an addendum to the core 3MF specification. Usage of this and any other 3MF extensions follow an a la carte model, defined in the core 3MF specification.

Part I, “3MF Documents,” presents the details of the primarily XML-based 3MF Document format. This section describes the XML markup that defines the composition of 3D documents and the appearance of each model within the document.

Part II, “Appendixes,” contains additional technical details and schemas too extensive to include in the main body of the text as well as convenient reference information.

The information contained in this specification is subject to change. Every effort has been made to ensure its accuracy at the time of publication.

This extension MUST be used only with Core specification 1.2.


## Document Conventions

Except where otherwise noted, syntax descriptions are expressed in the ABNF format as defined in RFC 4234.

Glossary terms are formatted like _this_.

Syntax descriptions and code are formatted as `Markdown code blocks.`

Replaceable items, that is, an item intended to be replaced by a value, are formatted in _`monospace cursive`_ type.

Notes are formatted as follows:

>**Note:** This is a note.

## Language Notes

In this specification, the words that are used to define the significance of each requirement are written in uppercase. These words are used in accordance with their definitions in RFC 2119, and their respective meanings are reproduced below:

- _MUST._ This word, or the adjective "REQUIRED," means that the item is an absolute requirement of the specification.
- _SHOULD._ This word, or the adjective "RECOMMENDED," means that there may exist valid reasons in particular circumstances to ignore this item, but the full implications should be understood and the case carefully weighed before choosing a different course.
- _MAY._ This word, or the adjective "OPTIONAL," means that this item is truly optional. For example, one implementation may choose to include the item because a particular marketplace or scenario requires it or because it enhances the product. Another implementation may omit the same item.

## Software Conformance

Most requirements are expressed as format or package requirements rather than implementation requirements.

In order for consumers to be considered conformant, they must observe the following rules:

- They MUST NOT report errors when processing conforming instances of the document format except when forced to do so by resource exhaustion.
- They SHOULD report errors when processing non-conforming instances of the document format when doing so does not pose an undue processing or performance burden.

In order for producers to be considered conformant, they must observe the following rules:

- They MUST NOT generate any new, non-conforming instances of the document format.
- They MUST NOT introduce any non-conformance when modifying an instance of the document format.

Editing applications are subject to all of the above rules.

# Part I. 3MF Documents


## Chapter 1. Overview of Additions

**add image**

This chapter describes new non-object resources. Each of these resources is OPTIONAL for producers but MUST be supported by consumers that specify support for this materials extension of 3MF.

As a general idea, the following resource groups will determine different ways of representing material properties of a part. The corresponding resource IDs MAY be referenced by triangle attributes defined in the core specification.

As there are existing file formats and use cases which need multiple pieces of information per triangle, it is possible to define multiple properties per triangle (see chapter 5). Consumers MUST be strict in obeying the mixing rules as laid out in the corresponding paragraphs to avoid ambiguous interpretation of the design intent.


### 1.1. Resources

All the new elements defined in this 3MF extension specification live under the <resources> element from the core 3MF specification. The <object> and <basematerials> elements are from the core spec, while the rest are defined in the following chapters. The ordering shown here is not enforced in the schema, as these extension elements all fall under the <any> element from the core spec.

### 1.2. sRGB and linear color values

The 3MF core specification (Chapter 5.1.1) mentions that whenever 3MF uses colors that are expressed as #RRGGBB hexadecimal quantities with 8 bits per color channel, they are assumed to be in sRGB color space. 3MF uses sRGB as specified by the World Wide Web Consortium (http://www.w3.org/Graphics/Color/sRGB).

Since human perception of brightness changes approximately with the logarithm of object's actual brightness, color data is usually encoded using a non-linear color component transfer. Such encoding is used to optimize the usage of bits, especially when individual R, G, B color channels are expressed as 8-bit quantities, as is the case with JPEG and PNG formats.

For inverse transformation to take place, it is necessary to obtain a normalized C_sRGB color triplet by dividing each channel by 255 (or 2^n-1, where n is the number of bits per channel):

    C_sRGB = {R_sRGB, G_sRGB, B_sRGB} = { #RR/255, #GG/255, #BB/255 }

Where C_sRGB is an sRGB color triplet not including the alpha channel, with elements normalized to [0, 1] range.

The inverse transformation from sRGB color space to linear space is defined as:

**add image**

This equation MUST be applied separately to each channel in the C_sRGB triplet to create the C_linear triplet.

    C_linear = {R_linear, G_linear, B_linear}

Where R_linear, G_linear, B_linear are separately calculated from the associated R_sRGB, G_sRGB, B_sRGB values using the above formula.

For this specification, this transformation is called the inverse color component transfer function. Unless specified otherwise, a client SHOULD perform a vertex color interpolation and a texture interpolation in sRGB, but apply the inverse color component transfer function to sRGB colors before multi-properties color blending takes place. Those blending operations SHOULD be performed in linear RGB space. This corresponds to common practices in Computer Graphics and hardware supported rendering operations.

The forward color component transfer function from linear to sRGB color space is defined as:

**add image**

This equation MUST be applied separately to each channel in the C_linear  triplet. sRGB values should be kept in the [0, 1] range while applying interpolations. To return sRGB to an 8-bit triplet (before persisting to disk, for example), it is necessary to multiply each channel by the required bits per channel (255, for 8-bit) and round to the nearest integer.

### 1.3. Material Gradients and Interpolation Methods

The 3MF core specification (Chapter 4.1.4: Triangles) describes properties e.g. color to be specified for each vertex of a triangle. Specifically, an sRGB triplet can be assigned to each vertex of a triangle. Color gradients within a triangle should be calculated by performing an interpolation in sRGB using a barycentric interpolation method. Performing color vertex interpolations in sRGB space corresponds to common practices in 2D and 3D imaging applications and is closer to an interpolation in a perceptual uniform space than an interpolation in a linear RGB space would be. 

### 1.4. Base Materials 

The 3MF core specification (Chapter 5: Material Resources) describes a base material type. This extension adds an additional attribute to the base material element representing display properties that allow realistic rendering of materials to a display.

Element **\<basematerials>**

![BaseMaterials](images/element_basematerials.png)

##### Attributes
| Name | Type | Use | Default | Annotation |
| --- | --- | --- | --- | --- |
| Attributes described in Core spec | ... | ... | ... | ... |
| displaypropertiesid | **ST_ResourceID** | optional | | Reference to a <displayproperties> element providing additional information about how to display the material on a device display |

## Chapter 2. Color Groups

Element **\<colorgroup>**

##### Attributes
| Name | Type | Use | Default | Annotation |
| --- | --- | --- | --- | --- |
| id | **ST_ResourceID** | required |  | Unique ID among all resources (which could include elements from extensions to the spec). |
| displaypropertiesid | **ST_ResourceID** | optional | | Reference to a <displayproperties> element providing additional information about how to display the material on a device display |
| @anyAttribute | | | | |
    
A <colorgroup> element acts as a container for color properties.
    
The order of the color elements forms an implicit 0-based index that is referenced by other elements, such as the <object> and <triangle> elements. 

A producer MAY define multiple <colorgroup> containers to help organize the file, for instance by grouping colors related to specific objects.
    
The displaypropertiesid attribute references a <displayproperties> group containing additional properties that describe how best to display a mesh with this material on a device display.
    
A <colorgroup> describes a set of surface color properties and SHOULD NOT reference translucent display properties. To achieve a translucent effect with surface color, a multi-properties group SHOULD be used instead. For more information, refer to Chapter 7: Display Properties Overview.

### 2.1. Color

Element **\<color>**

##### Attributes
| Name | Type | Use | Default | Annotation |
| --- | --- | --- | --- | --- |
| color | **ST_ColorValue** | required |  | Specifies the sRGB color for rendering the material. |

Colors are used to represent rich color, specifically what most 3D formats call “vertex colors”. These elements are used when color is the only property of interest for the material, and a large number will be needed. The format is the same sRGB color as defined in the core 3MF specification.

Colors are assumed to be fully opaque (alpha = #FF) except when used as a non-base layer inside a <multiproperties> element.	
To avoid integer overflows, a color group MUST contain less than 2^31 colors.

## Chapter 3. Texture 2D Groups

Element **\<texture2dgroup>**

##### Attributes
| Name | Type | Use | Default | Annotation |
| --- | --- | --- | --- | --- |
| id | **ST_ResourceID** | required |  | Unique ID among all resources (which could include elements from extensions to the spec). |
| texid | **ST_ResourceID** | required |  | Reference to the <texture2d> element with the matching id attribute value. |
| displaypropertiesid | **ST_ResourceID** | optional | | Reference to a <displayproperties> element providing additional information about how to display the material on a device display |
| @anyAttribute | | | | |
    
A <texture2dgroup> element acts as a container for texture coordinate properties. The order of these elements forms an implicit 0-based index that is referenced by other elements, such as the <object> and <triangle> elements. It also specifies which image to use, via texid. The referenced <texture2d> elements are described below in Chapter 6.

The texture’s alpha channel is assumed to be fully opaque (alpha = #FF) unless specified otherwise.

The displaypropertiesid attribute references a <displayproperties> group containing additional properties that describe how best to display a mesh with this material on a device display. A <texture2Dgroup> describes a set of surface color properties and MUST NOT reference translucent display properties. To achieve a translucent effect through a texture, a multi-properties group MUST be used instead. For more information, refer to Chapter 7: Display Properties Overview.

### 3.1. Texture 2D Coordinate

Element **\<tex2coord>**

##### Attributes
| Name | Type | Use | Default | Annotation |
| --- | --- | --- | --- | --- |
| u | **ST_Number** | required |  | The u-coordinate within the texture, horizontally right from the origin in the lower left of the texture. |
| v | **ST_Number** | required |  | The v-coordinate within the texture, vertically up from the origin in the lower left of the texture. |
| @anyAttribute | | | | |

Texture coordinates map a vertex of a triangle to a position in image space (U, V coordinates). Texture mapping allows high-resolution color bitmaps to be applied to any surface. The primary advantage of texture mapping over the vertex colors of the previous section is that the textures allow color at a much finer detail level than the underlying mesh, while vertex colors are always at the same resolution as the mesh.

To avoid integer overflows, a texture coordinate group MUST contain less than 2^31 tex2coords.

## Chapter 4. Composite Materials

Element **\<compositematerials>**

##### Attributes
| Name | Type | Use | Default | Annotation |
| --- | --- | --- | --- | --- |
| id | **ST_ResourceID** | required |  | Unique ID among all resources (which could include elements from extensions to the spec). |
| matid | **ST_ResourceID** | required |  | Reference to the base material group element with the matching id attribute value (e.g. <basematerials>). |
| matindices | **ST_ResourceIndices** | required |  | A space-delimited list of ST_ResourceIndex values of the material constituents |
| displaypropertiesid | **ST_ResourceID** | optional |  | Reference to a <displayproperties> element providing additional information about how to display the material on a device display |
| @anyAttribute | | | | |

##### Elements
| Name | Type | Use | Default | Annotation |
| --- | --- | --- | --- | --- |
| composite | **CT_Composite** | required |   |   |

A <compositematerials> element acts as a container for composite materials. The order of these elements forms an implicit 0-based index that is referenced by other elements, such as the <object> and <triangle> elements. A producer MAY define multiple <compositematerials> containers, for instance by grouping mixtures of different materials.

The <compositematerials> element defines materials derived by mixing 2 or more base materials in defined ratios. This collective mixture is referred to as a composite material. The matid attribute specifies the material group that all constituents are from, which MUST be a <basematerials> group. The matindices attribute specifies the indices of the materials to mix.

The displaypropertiesid attribute references a <displayproperties> group containing additional properties that describe how best to display the material when previewing a mesh with this material on a device display. For more information, refer to Chapter 7: Display Properties Overview.

### 4.1. Composite

Element **\<composite>**

##### Attributes
| Name | Type | Use | Default | Annotation |
| --- | --- | --- | --- | --- |
| Values | **ST_Numbers** | required |  | A space-delimited list of ST_Number values between 0 and 1, inclusive representing the fraction of the material constituents, respectively. |
| @anyAttribute | | | | |

attributes	Name	Type	Use	Default	Fixed	Annotation
Values	ST_Numbers	required			A space-delimited list of ST_Number values between 0 and 1, inclusive representing the fraction of the material constituents, respectively.
@anyAttribute					

The <composite> element defines a values attribute, which specifies the proportion of the overall mixture for each material. If the sum of the values is greater than zero, consumers MUST divide each value by the sum of the values of all constituent value attributes to apply the correct proportion for each material. If the sum of all constituent value attributes is zero, each value MUST be treated as 1.0 divided by the number of constituent elements.
    
If the values list is shorter than the matindices list, consumers MUST use a default value of zero for unspecified values. Extra values MUST be ignored.

To avoid integer overflows, a composite group MUST contain less than 2^31 composites.

## Chapter 5. Multi-Properties

Element **\<multiproperties>**

##### Attributes
| Name | Type | Use | Default | Annotation |
| --- | --- | --- | --- | --- |
| id | **ST_ResourceID** | required |  | Unique ID among all resources (which could include elements from extensions to the spec). |
| pids | **ST_ResourceID** | required |  | A space-delimited list of ST_ResourceID values representing the property group of each constituent |
| blendmethods | **ST_BlendMethods** | optional | mix | Defines the list of equation(s) to use when blending each layer with the previous layer: “mix” or “multiply”. One value should be specified for each layer minus the first layer which is ignored. |
| @anyAttribute | | | | |

##### Elements
| Name | Type | Use | Default | Annotation |
| --- | --- | --- | --- | --- |
| multi | **CT_Multi** | required |   |   |


A <multiproperties> element acts as a container for <multi> elements. The order of these elements forms an implicit 0-based index that is referenced by other elements, such as the <object> and <triangle> elements. The pids list MUST NOT contain more than one reference to a material (base or composite). The pids list MUST NOT contain more than one reference to a colorgroup (for performance reasons). The pids list MUST NOT contain any references to a multiproperties. A producer MAY define multiple <multiproperties> containers, for instance to layer textures in a different order or to specify a different material.
    
A material, if included, MUST be positioned as the first layer, with color information – texture or colors, in subsequent layers. This arrangement describes the composition of an object by defining the “shell” on top of which the other layers in the multi-properties are blended.

First, the properties are independently sampled and linearly interpolated on a triangle, then layered using the order specified within the pids attribute. To determine the resulting color, the individual contributions of all layers are accumulated by considering their opacity and blending mode. When a layer is processed, it is blended with the already accumulated result of previous blending operations, forming a new accumulated value. For each blending mode, two equations are provided. One is used to accumulate RGB color and the second one is used to accumulate opacity.

The blendmethods attribute allows the producer to specify the equation to use when blending the colors between two layers. The blendmethods attribute provides a list of “mix” or “multiply” values associated with each layer in the group describing how to be blended with the previous layer results. The first layer MUST be omitted. If there are more layers than blendmethods values + 1 specified in the list, “mix” is assumed to be the default operation. There MUST NOT be more blendmethods than layers – 1.
The initial accumulated alpha value, as well as the first layer opacity, is assumed to be #FF (fully opaque). The initial accumulated RGB value is copied from the first layer and the process of blending starts with the 2nd layer and continues until all subsequent layers are processed.

If the first layer contains a material with translucent display property, base material, or composite material, it is skipped (including the first entry in the <blendmethods> list). The 2nd layer’s RGB is used to initialize the accumulated RGB color and alpha is initialized depending on the blending mode:
    
•	#FF (fully opaque) for “multiply” blending mode.
•	2nd layer’s actual alpha for “mix” blending mode.

Blending starts with the 3rd layer, in this case. Once the blend is determined, the resulting RGB color is applied to the material specified in the first layer using the accumulated alpha value as opacity.

For example, if the accumulated alpha value indicates 50% opacity, it implies that color is applied in such way that 50% of the underlying surface shows through.

Linear “mix” interpolation would be represented as:

    accumulatedColor.rgb = newLayer.rgb * newLayer.a + accumulatedColor.rgb * (1 – newLayer.a)
    accumulatedColor.a = newLayer.a + accumulatedColor.a * (1 – newLayer.a)

Multiplication would be represented as:

    accumulatedColor.rgb = newLayer.rgb * accumulatedColor.rgb
    accumulatedColor.a = newLayer.a * accumulatedColor.a

Note: Users coming from a Graphic Arts background who prefer color blending to be performed in sRGB or any other color space are advised to perform the composition in a 2D imaging application and then apply the blended 2D textures to an object.

Consider the following example:

We want to apply this texture containing alpha channel values indicating transparency (grey color) to <triangle> elements.

**image**

The result should look like this (a 3MF sample is provided in Appendix C.2.):

**image**

And not like this:

**image**

To achieve this affect, multi-properties will be used where the first layer contains material with a translucent displayproperties. The underlying material MAY be rendered translucent to allow underlying model material to “show through” transparent areas. The subsequent layers, which can be of any type, are then blended according to the method described above.

For example, assume a second layer is blended on top of the first layer containing a color value with a 50% alpha.
A display renderer easily determines the translucent surface color based on object’s attenuation, thickness and background lighting. Whatever color that turns out to be is blended with 1st layer’s color in a separate rendering pass. 

It is different for a 3d printer. If we apply opaque color over a transparent layer, the result depends on what we mean by 50% alpha. If we blend two opaque colors, it’s easy to determine their average and apply paint because it’s still opaque. 

In this case, the alpha channel MUST describe color opacity. Fully opaque alpha means that the underlying translucent surface doesn’t show through. Zero alpha means that the underlying surface fully shows through. 50% alpha allows 50% of background light to pass. 
Imagine the surface as a set of infinitesimally small micro-facets, half of the facets would be covered in fully opaque paint and the other half would show through. Like spray paint which only has half the normal throttle. This behavior is fully consistent with the display renderer described above and it works for standard blending mode.

When physically printing, a material and display properties MAY be ignored. But when rendering on screen, the display color and display properties SHOULD be blended to provide a realistic preview.

### 5.1. Multi

Element **\<multi>**

##### Attributes
| Name | Type | Use | Default | Annotation |
| --- | --- | --- | --- | --- |
| pindices | **ST_ResourceIndices** | required |  | A space-delimited list of ST_ResourceIndex values of the constituents |
| @anyAttribute | | | | |

If the pindices list is shorter than the pids list, consumers MUST use a default index of zero for any unspecified pindices. Extra pindices MUST be ignored. To avoid integer overflows, a multi property group MUST contain less than 2^31 elements.

## Chapter 6. Texture 2d

Element **\<texture2d>**

##### Attributes
| Name | Type | Use | Default | Annotation |
| --- | --- | --- | --- | --- |
| id | **ST_ResourceID** | required |  | Specifies a unique identifier for this texture resource. |
| path | **ST_UriReference** | required |  | Specifies the part name of the texture data. |
| contenttype | **ST_ContentType** | required |  | Specifies the content type of the 2D Texture part referenced by the path attribute. Valid values are image/jpeg and image/png. |
| tilestyleu | **ST_TileStyle** |  | wrap | Specifies how tiling should occur in the u axis in order to fill the overall requested area. Valid values are wrap, mirror, clamp, none. |
| tilestylev | **ST_TileStyle** |  | wrap | Specifies how tiling should occur in the v axis in order to fill the overall requested area. Valid values are wrap, mirror, clamp, none. |
| filter | **ST_Filter** |  | auto | Specifies the texture filter to apply when scaling the source texture.  Allowed values are “auto”, “linear”, “nearest” |
| @anyAttribute | | | | |

A 2D texture resource provides information about texture image data, found via the provided path reference, which MUST also be the target of a 3D Texture relationship from the 3D Model part.

The following table shows the logical interpretation of possible input pixel layouts. The meaning of symbols is as follows: R – red, G – green, B – blue, A – alpha, Y – grayscale.

For example, if the specification says that a certain value is sampled from the texture’s R channel, but the referenced texture is only monochromatic then grayscale channel is interpreted as the R color channel. Similarly, color values sampled from a monochromatic texture are interpreted as if all R, G, B color channels shared the same grayscale value.

Logical interpretation as a RGBA value:

| Input pixel layout | | | | |
| --- | --- | --- | --- | --- |
| RGBA* | R | G | B | A |
| RGB | R | G | B | #FF |
| YA* | Y | Y | Y | A |
| Y | Y | Y | Y | #FF |
* *These pixel layouts are only supported by the PNG format.

If there is no alpha channel present in the texture, the default value #FF SHOULD be used. Unless specified otherwise, alpha channel is assumed to be in linear space while color and grayscale channels are assumed to be in sRGB color space. Texture filtering should be performed in sRGB, but a client SHOULD perform conversion to linear RGB (see Chapter 1.2.) before linear operations such as multi property blending take place.

The box attribute was DEPRECATED in version 1.2. Producers SHOULD NOT generate it and consumer SHOULD ignore it.

tilestyleu, tilestylev - The tile style of wrap essentially means that the same texture SHOULD be repeated in the specified axis (both in the positive and negative directions), for the axis value. The tile style of mirror means that each time the texture width or height is exceeded, the next repetition of the texture SHOULD be reflected across a plane perpendicular to the axis in question. The tile style of clamp means all Texture 2D Coordinates outside of the range zero to one will be assigned the color of the nearest edge pixel. The tile style of none means that all Texture 2D Coordinates outside the range zero to one will be assigned the color of the default object color. If the default object color is not defined the choice for the color is left to the consumer.

The only supported content types are JPEG and PNG, as more specifically specified in the 3MF core spec under the Thumbnails section.
filter - The producer MAY require the use of a specific filter type by specifying either “linear” for bilinear interpolation or “nearest” for nearest neighbor interpolation. The producer SHOULD use “auto” to indicate to the consumer to use the highest quality filter available. If source texture is scaled with the model, the specified filter type MUST be applied to the scaling operation. The default value is “auto”.

The following example shows how the filter MUST be applied to the texture. Figure 6-1 shows an example of a small texture which is tiled by vertically mirroring and horizontally wrapping. It illustrates that that the texture pixels are located at the center of each cell. All the filter operations should be performed in sRGB.

*Figure 6-1:  The image and tiling used as example, showing where the texture pixels are located.

**image**

Finally, Figure 6-2 shows the nearest and the linear filters output by filling the cells.
*Figure 6-2:  Texture filtering of a 3 x 2 image (Figure 1a) with tilestyleu=WRAP and tilestylev=MIRROR.

**image**

## Chapter 7. Display Properties Overview

Display properties contain extra information about the material to help how to describe it so that a material can be realistically and consistently rendered on the screen. For example, a metal material will be highly shiny and reflective, where a translucent material will allow light to pass through. This information is useful mainly for display purposes.

Physically based rendering (PBR) is an approach to real-time rendering of materials that delivers physically plausible surface reflections in a variety of lighting conditions. It can be viewed as an extension of color where the surface is defined also by its specular reflectance and roughness.

Most real-world materials fall into two categories:

1.	Non-metals (dielectrics). Materials like plastics or ceramics tend to be less reflective and the light that penetrates the surface is usually scattered and reemitted back into the environment. Because of that their diffuse reflectance can be extremely high while their specular reflectance is usually only around 4%. Specular reflections are usually uncolored.

2.	Metals (conductors). These materials tend to be very reflective and they usually absorb rather than scatter any light that happens to penetrate the surface. Therefore, their specular reflectance is extremely high and their diffuse reflectance approaches zero. Specular reflections can be color tinted, e.g. in case of gold and copper. 

Because of this duality, some systems adopted the so-called metallic workflow in which the material is defined by its surface color and the degree to which it behaves as a metal. Another widely adopted approach is the so-called specular workflow in which both the diffuse reflectance and the specular reflectance are defined explicitly as sRGB color triplets. 

Both workflows use an additional parameter that specifies surface roughness, or its complimentary value called shininess (or smoothness). In this document, the terms ‘metallic workflow’ and ‘metallic representation’ as well as ‘specular workflow’ and ‘specular representation’ are used interchangeably.

For both workflows 3MF defines both “plain” and “textured” representation of the material.

Physically based materials specify only the appearance of material at the surface of the object. They do not describe the distribution of the material through the volume of the object. Similarly, they do not describe the chemical composition of the material.

Translucent materials have the quality of allowing light to pass through, unlike opaque materials which reflect some portions of electromagnetic spectrum while absorbing others. The portion of light that is not reflected into the environment is refracted and gradually absorbed as it travels through the material. Object thickness plays a significant role in how much light is absorbed. While transparent materials only affect the amount of light they let through, translucent ones can even alter its path, resulting in more diffuse appearance. Such appearance can be attributed to a combination of two factors – surface scattering due to object’s surface roughness and volume scattering due to microscopic non-uniformities in the material. In the latter case, the light does not follow a straight path but bounces many times inside the object before it is absorbed or reemitted somewhere else.

Display properties are represented by these five types – specular, metallic, specular with texture, metallic with texture, and translucent.

“metallic”, “specular”, and “translucent” types are only valid for <basematerials>, <compositematerials> and <colorgroup>. Where “metallictexture” and “speculartexture” are only valid for <texture2dgroup>.
    
The properties defined on a triangle that are from a display properties group MUST NOT form gradients, as interpolation between physically based materials is not defined in this specification. A consumer MUST apply the p1 property to the entire triangle. Properties p2 and p3 MUST be either unspecified or they MUST be equal to p1.

### 7.1. Specular Display Properties

Element **\<pbspeculardisplayproperties>**

#### Attributes
| Name | Type | Use | Default | Annotation |
| --- | --- | --- | --- | --- |
| id | **ST_ResourceID** | required |  | Unique ID among all resources (which could include elements from extensions to the spec). |
| @anyAttribute | | | | |
    

##### Elements
| Name | Type | Use | Default | Annotation |
| --- | --- | --- | --- | --- |
| pbspecular | **CT_PBSpecular** | required |   |   |

The <pbspeculardisplayproperties> are located under <resources> and contain a set of properties describing how to realistically display a specular material. They are optionally associated with specific materials through a “displaypropertiesid” attribute.

<pbspeculardisplayproperties> is a container for one or more <pbspecular> elements.
    
The order and count of the elements forms an implicit 0-based index in the same as the order and count of elements of the associated material group. For example, if a <basematerials> group includes a “displaypropertiesid” attribute pointing to a <pbspeculardisplayproperties> element, there will be the same number of <pbspecular> elements as <basematerial> elements where the first <pbspecular> element describes the first <basematerial> in the group.

### 7.1.1. Specular

Element **\<pbspecular>**

#### Attributes
| Name | Type | Use | Default | Annotation |
| --- | --- | --- | --- | --- |
| name | **xs:string** | required |  | Specifies the material name |
| specularcolor | **ST_ColorValue** |   | #383838 | Specular reflectance value |
| glossiness | **ST_Number** |   | 0 | Surface glossiness (smoothness) value |
| @anyAttribute | | | | |
    
The <pbspecular> element infers a diffuse color from the color attribute of the material it is associated with. For example, when <pbspecular> display properties are associated with a <basematerial>, the “displaycolor” attribute from basematerial specifies a diffuse color to apply using pbspecular display properties. Similarly, when <pbspecular> display properties are associated with a <color> material, the “color” attribute specifies the diffuse color to apply.
 
The diffuse color describes the surface color. It is an sRGB color triplet that specifies diffuse reflectance of the surface. It represents the proportion of light which is reflected off the surface in diffuse fashion in respective red, green and blue wavelength regions. Diffuse reflection is an idealized concept in which the incident light scatters in all directions independently of the angle at which it arrives. 
    
In order to obtain RGB coefficients in the 0..1 range that can be used in lighting calculations, the inverse color component transfer function (see  Chapter 1.2. sRGB and linear RGB color values) MUST be applied to convert colors from sRGB color space to linear RGB space.

**Name**
    
Material name is intended to convey design intent. Producers SHOULD avoid machine-specific naming in favor of more portable descriptions. 

**Specularcolor**

Specular color is a sRGB color triplet that specifies specular reflectance of the surface at normal incidence (the condition in which the light beam is perpendicular to the surface). It represents the proportion of light which is reflected off the surface in mirror-like fashion in respective red, green and blue wavelength regions. Unlike diffuse reflection, specular reflection depends on the position of the observer and the angle of incidence. Intuitively, the parameter describes the intensity and color tint of surface reflections.

The default value #383838 corresponds to a linear specular reflectance value of (0.04, 0.04, 0.04) common for plastics and other dielectric materials.

In order to obtain linear RGB coefficients in the 0..1 range that can be used for lighting calculations, the inverse color component transfer function (see  Chapter 1.2. sRGB and linear RGB color values) MUST be applied.

**Glossiness**

Glossiness is a scalar surface property in 0..1 range that specifies how smooth the surface is. Real-world surfaces have microscopic imperfections which can cause scattering of incident light. Because these imperfections are beyond the resolution of common 3D printers and displays, it is assumed that their scattering properties can be modeled statistically using micro-facet surface model (see Appendix D. Micro-facet Surface Model and BRDF) in which the surface consists of infinitesimally small, mirror-like facets which only reflect light in a single direction according to their orientation (normal). A value of 1 means that the surface is ideally smooth, with micro-facet normals oriented the same way as the surface normal. A value of 0 means a very rough surface for which the distribution of micro-facet normals is a uniform hemisphere. 

A consumer SHOULD follow the GLTF specified behavior for determining surface reflectance properties from the roughness / glossiness material parameter to obtain consistent visual results. For more information, refer to Appendix D. Micro-facet Surface Model and BRDF or to the GLTF 2.0 specification Appendix B: BRDF Implementation.

### 7.2. Metallic Display Properties

