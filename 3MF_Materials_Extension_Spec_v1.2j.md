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




