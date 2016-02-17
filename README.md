Building your repository and other useful commands
./gradlew clean build will build your project (and run unit tests)

./gradlew clean build installPackage will build and install your project

./gradlew test will run your unit tests

./gradlew idea will generate IntelliJ project files (after that you can simply open this repository in IntelliJ)

./gradlew clean build installPackage -Dhost=(install host) -Dport=(install port) will build and install your project to a specific host and port that you define in the command. The default values if you do not set these variables are localhost for host and 4502 for port.

 Create a custom Replication Agent for Solr for Indexing Data to Solr on Replication

 1. Transport URI as localhost:8983/solr/solrpoc?commit=true
 2. Replication Agent as  POC Solr XML Serializer


Use this schema.xml

<?xml version="1.0" encoding="UTF-8" ?>
<!--
 Licensed to the Apache Software Foundation (ASF) under one or more
 contributor license agreements.  See the NOTICE file distributed with
 this work for additional information regarding copyright ownership.
 The ASF licenses this file to You under the Apache License, Version 2.0
 (the "License"); you may not use this file except in compliance with
 the License.  You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
-->


<schema name="example" version="1.5">
    <!-- If you remove this field, you must _also_ disable the update log in solrconfig.xml
       or Solr won't start. _version_ and update log are required for SolrCloud
    -->
    <field name="_version_" type="long" indexed="true" stored="true"/>

    <!-- points to the root document of a block of nested documents. Required for nested
       document support, may be removed otherwise
    -->
    <field name="_root_" type="string" indexed="true" stored="false"/>

    <!-- Only remove the "id" field if you have a very good reason to. While not strictly
      required, it is highly recommended. A <uniqueKey> is present in almost all Solr
      installations. See the <uniqueKey> declaration below where <uniqueKey> is set to "id".
    -->
    <field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false" />

    <field name="title" type="text_general" indexed="true" stored="true" multiValued="true"/>
    <field name="description" type="text_general" indexed="true" stored="true"/>
    <field name="keywords" type="text_general" indexed="true" stored="true"/>
    <field name="category" type="text_general" indexed="true" stored="true"/>
    <field name="content_type" type="string" indexed="true" stored="true" multiValued="true"/>
    <field name="last_modified" type="date" indexed="true" stored="true"/>

    <field name="date_created" type="date" indexed="true" stored="true"/>
    <field name="tags" type="string" indexed="true" stored="true" multiValued="true"/>
    <field name="template" type="string" indexed="true" stored="true"/>
    <field name="hide_from_search" type="boolean" indexed="true" stored="true"/>
    <!-- Main body of document extracted by SolrCell.
         NOTE: This field is not indexed by default, since it is also copied to "text"
         using copyField below. This is to save space. Use this field for returning and
         highlighting document content. Use the "text" field to search the content. -->
    <field name="content" type="text_general" indexed="false" stored="true" multiValued="true"/>
    <field name="archived" type="text_general" indexed="true" stored="true"/>


    <!-- catchall field, containing all other searchable text fields (implemented
         via copyField further on in this schema  -->
    <field name="text" type="text_general" indexed="true" stored="false" multiValued="true"/>

    <!-- catchall text field that indexes tokens both normally and in reverse for efficient
         leading wildcard queries. -->
    <field name="text_rev" type="text_general_rev" indexed="true" stored="false" multiValued="true"/>

    <dynamicField name="*_i"  type="int"    indexed="true"  stored="true"/>
    <dynamicField name="*_is" type="int"    indexed="true"  stored="true"  multiValued="true"/>
    <dynamicField name="*_s"  type="string"  indexed="true"  stored="true" />
    <dynamicField name="*_ss" type="string"  indexed="true"  stored="true" multiValued="true"/>
    <dynamicField name="*_l"  type="long"   indexed="true"  stored="true"/>
    <dynamicField name="*_ls" type="long"   indexed="true"  stored="true"  multiValued="true"/>
    <dynamicField name="*_t"  type="text_general"    indexed="true"  stored="true"/>
    <dynamicField name="*_txt" type="text_general"   indexed="true"  stored="true" multiValued="true"/>
    <dynamicField name="*_en"  type="text_en"    indexed="true"  stored="true" multiValued="true"/>
    <dynamicField name="*_b"  type="boolean" indexed="true" stored="true"/>
    <dynamicField name="*_bs" type="boolean" indexed="true" stored="true"  multiValued="true"/>
    <dynamicField name="*_f"  type="float"  indexed="true"  stored="true"/>
    <dynamicField name="*_fs" type="float"  indexed="true"  stored="true"  multiValued="true"/>
    <dynamicField name="*_d"  type="double" indexed="true"  stored="true"/>
    <dynamicField name="*_ds" type="double" indexed="true"  stored="true"  multiValued="true"/>

    <!-- Type used to index the lat and lon components for the "location" FieldType -->
    <dynamicField name="*_coordinate"  type="tdouble" indexed="true"  stored="false" />

    <dynamicField name="*_dt"  type="date"    indexed="true"  stored="true"/>
    <dynamicField name="*_dts" type="date"    indexed="true"  stored="true" multiValued="true"/>
    <dynamicField name="*_p"  type="location" indexed="true" stored="true"/>

    <!-- some trie-coded dynamic fields for faster range queries -->
    <dynamicField name="*_ti" type="tint"    indexed="true"  stored="true"/>
    <dynamicField name="*_tl" type="tlong"   indexed="true"  stored="true"/>
    <dynamicField name="*_tf" type="tfloat"  indexed="true"  stored="true"/>
    <dynamicField name="*_td" type="tdouble" indexed="true"  stored="true"/>
    <dynamicField name="*_tdt" type="tdate"  indexed="true"  stored="true"/>

    <dynamicField name="*_c"   type="currency" indexed="true"  stored="true"/>

    <dynamicField name="ignored_*" type="ignored" multiValued="true"/>
    <dynamicField name="attr_*" type="text_general" indexed="true" stored="true" multiValued="true"/>

    <dynamicField name="random_*" type="random" />

    <!-- uncomment the following to ignore any fields that don't already match an existing
         field name or dynamic field, rather than reporting them as an error.
         alternately, change the type="ignored" to some other type e.g. "text" if you want
         unknown fields indexed and/or stored by default -->
    <!--dynamicField name="*" type="ignored" multiValued="true" /-->




    <!-- Field to use to determine and enforce document uniqueness.
         Unless this field is marked with required="false", it will be a required field
      -->
    <uniqueKey>id</uniqueKey>

    <copyField source="title" dest="text"/>
    <copyField source="description" dest="text"/>
    <copyField source="keywords" dest="text"/>
    <copyField source="content" dest="text"/>
    <copyField source="content_type" dest="text"/>
    <copyField source="tags" dest="text"/>
    <copyField source="template" dest="text"/>

    <fieldType name="string" class="solr.StrField" sortMissingLast="true" />

    <!-- boolean type: "true" or "false" -->
    <fieldType name="boolean" class="solr.BoolField" sortMissingLast="true"/>
    <fieldType name="int" class="solr.TrieIntField" precisionStep="0" positionIncrementGap="0"/>
    <fieldType name="float" class="solr.TrieFloatField" precisionStep="0" positionIncrementGap="0"/>
    <fieldType name="long" class="solr.TrieLongField" precisionStep="0" positionIncrementGap="0"/>
    <fieldType name="double" class="solr.TrieDoubleField" precisionStep="0" positionIncrementGap="0"/>

    <fieldType name="tint" class="solr.TrieIntField" precisionStep="8" positionIncrementGap="0"/>
    <fieldType name="tfloat" class="solr.TrieFloatField" precisionStep="8" positionIncrementGap="0"/>
    <fieldType name="tlong" class="solr.TrieLongField" precisionStep="8" positionIncrementGap="0"/>
    <fieldType name="tdouble" class="solr.TrieDoubleField" precisionStep="8" positionIncrementGap="0"/>

    <fieldType name="date" class="solr.TrieDateField" precisionStep="0" positionIncrementGap="0"/>

    <!-- A Trie based date field for faster date range queries and date faceting. -->
    <fieldType name="tdate" class="solr.TrieDateField" precisionStep="6" positionIncrementGap="0"/>


    <!--Binary data type. The data should be sent/retrieved in as Base64 encoded Strings -->
    <fieldtype name="binary" class="solr.BinaryField"/>

    <fieldType name="random" class="solr.RandomSortField" indexed="true" />

    <!-- A text field that only splits on whitespace for exact matching of words -->
    <fieldType name="text_ws" class="solr.TextField" positionIncrementGap="100">
        <analyzer>
            <tokenizer class="solr.WhitespaceTokenizerFactory"/>
        </analyzer>
    </fieldType>

    <!-- A text type for English text where stopwords and synonyms are managed using the REST API -->
    <fieldType name="managed_en" class="solr.TextField" positionIncrementGap="100">
        <analyzer>
            <tokenizer class="solr.StandardTokenizerFactory"/>
            <filter class="solr.ManagedStopFilterFactory" managed="english" />
            <filter class="solr.ManagedSynonymFilterFactory" managed="english" />
        </analyzer>
    </fieldType>

    <!-- A general text field that has reasonable, generic
         cross-language defaults: it tokenizes with StandardTokenizer,
	 removes stop words from case-insensitive "stopwords.txt"
	 (empty by default), and down cases.  At query time only, it
	 also applies synonyms. -->
    <fieldType name="text_general" class="solr.TextField" positionIncrementGap="100">
        <analyzer type="index">
            <tokenizer class="solr.StandardTokenizerFactory"/>
            <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
            <!-- in this example, we will only use synonyms at query time
            <filter class="solr.SynonymFilterFactory" synonyms="index_synonyms.txt" ignoreCase="true" expand="false"/>
            -->
            <filter class="solr.LowerCaseFilterFactory"/>
        </analyzer>
        <analyzer type="query">
            <tokenizer class="solr.StandardTokenizerFactory"/>
            <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
            <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
            <filter class="solr.LowerCaseFilterFactory"/>
        </analyzer>
    </fieldType>

    <!-- A text field with defaults appropriate for English: it
         tokenizes with StandardTokenizer, removes English stop words
         (lang/stopwords_en.txt), down cases, protects words from protwords.txt, and
         finally applies Porter's stemming.  The query time analyzer
         also applies synonyms from synonyms.txt. -->
    <fieldType name="text_en" class="solr.TextField" positionIncrementGap="100">
        <analyzer type="index">
            <tokenizer class="solr.StandardTokenizerFactory"/>
            <!-- in this example, we will only use synonyms at query time
            <filter class="solr.SynonymFilterFactory" synonyms="index_synonyms.txt" ignoreCase="true" expand="false"/>
            -->
            <!-- Case insensitive stop word removal.
            -->
            <filter class="solr.StopFilterFactory"
                    ignoreCase="true"
                    words="lang/stopwords_en.txt"
                    />
            <filter class="solr.LowerCaseFilterFactory"/>
            <filter class="solr.EnglishPossessiveFilterFactory"/>
            <filter class="solr.KeywordMarkerFilterFactory" protected="protwords.txt"/>
            <!-- Optionally you may want to use this less aggressive stemmer instead of PorterStemFilterFactory:
                <filter class="solr.EnglishMinimalStemFilterFactory"/>
            -->
            <filter class="solr.PorterStemFilterFactory"/>
        </analyzer>
        <analyzer type="query">
            <tokenizer class="solr.StandardTokenizerFactory"/>
            <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
            <filter class="solr.StopFilterFactory"
                    ignoreCase="true"
                    words="lang/stopwords_en.txt"
                    />
            <filter class="solr.LowerCaseFilterFactory"/>
            <filter class="solr.EnglishPossessiveFilterFactory"/>
            <filter class="solr.KeywordMarkerFilterFactory" protected="protwords.txt"/>
            <!-- Optionally you may want to use this less aggressive stemmer instead of PorterStemFilterFactory:
                <filter class="solr.EnglishMinimalStemFilterFactory"/>
            -->
            <filter class="solr.PorterStemFilterFactory"/>
        </analyzer>
    </fieldType>

    <!-- A text field with defaults appropriate for English, plus
	 aggressive word-splitting and autophrase features enabled.
	 This field is just like text_en, except it adds
	 WordDelimiterFilter to enable splitting and matching of
	 words on case-change, alpha numeric boundaries, and
	 non-alphanumeric chars.  This means certain compound word
	 cases will work, for example query "wi fi" will match
	 document "WiFi" or "wi-fi".
        -->
    <fieldType name="text_en_splitting" class="solr.TextField" positionIncrementGap="100" autoGeneratePhraseQueries="true">
        <analyzer type="index">
            <tokenizer class="solr.WhitespaceTokenizerFactory"/>
            <!-- in this example, we will only use synonyms at query time
            <filter class="solr.SynonymFilterFactory" synonyms="index_synonyms.txt" ignoreCase="true" expand="false"/>
            -->
            <!-- Case insensitive stop word removal.
            -->
            <filter class="solr.StopFilterFactory"
                    ignoreCase="true"
                    words="lang/stopwords_en.txt"
                    />
            <filter class="solr.WordDelimiterFilterFactory" generateWordParts="1" generateNumberParts="1" catenateWords="1" catenateNumbers="1" catenateAll="0" splitOnCaseChange="1"/>
            <filter class="solr.LowerCaseFilterFactory"/>
            <filter class="solr.KeywordMarkerFilterFactory" protected="protwords.txt"/>
            <filter class="solr.PorterStemFilterFactory"/>
        </analyzer>
        <analyzer type="query">
            <tokenizer class="solr.WhitespaceTokenizerFactory"/>
            <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
            <filter class="solr.StopFilterFactory"
                    ignoreCase="true"
                    words="lang/stopwords_en.txt"
                    />
            <filter class="solr.WordDelimiterFilterFactory" generateWordParts="1" generateNumberParts="1" catenateWords="0" catenateNumbers="0" catenateAll="0" splitOnCaseChange="1"/>
            <filter class="solr.LowerCaseFilterFactory"/>
            <filter class="solr.KeywordMarkerFilterFactory" protected="protwords.txt"/>
            <filter class="solr.PorterStemFilterFactory"/>
        </analyzer>
    </fieldType>

    <!-- Less flexible matching, but less false matches.  Probably not ideal for product names,
         but may be good for SKUs.  Can insert dashes in the wrong place and still match. -->
    <fieldType name="text_en_splitting_tight" class="solr.TextField" positionIncrementGap="100" autoGeneratePhraseQueries="true">
        <analyzer>
            <tokenizer class="solr.WhitespaceTokenizerFactory"/>
            <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="false"/>
            <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_en.txt"/>
            <filter class="solr.WordDelimiterFilterFactory" generateWordParts="0" generateNumberParts="0" catenateWords="1" catenateNumbers="1" catenateAll="0"/>
            <filter class="solr.LowerCaseFilterFactory"/>
            <filter class="solr.KeywordMarkerFilterFactory" protected="protwords.txt"/>
            <filter class="solr.EnglishMinimalStemFilterFactory"/>
            <!-- this filter can remove any duplicate tokens that appear at the same position - sometimes
                 possible with WordDelimiterFilter in conjuncton with stemming. -->
            <filter class="solr.RemoveDuplicatesTokenFilterFactory"/>
        </analyzer>
    </fieldType>

    <!-- Just like text_general except it reverses the characters of
	 each token, to enable more efficient leading wildcard queries. -->
    <fieldType name="text_general_rev" class="solr.TextField" positionIncrementGap="100">
        <analyzer type="index">
            <tokenizer class="solr.StandardTokenizerFactory"/>
            <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
            <filter class="solr.LowerCaseFilterFactory"/>
            <filter class="solr.ReversedWildcardFilterFactory" withOriginal="true"
                    maxPosAsterisk="3" maxPosQuestion="2" maxFractionAsterisk="0.33"/>
        </analyzer>
        <analyzer type="query">
            <tokenizer class="solr.StandardTokenizerFactory"/>
            <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
            <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
            <filter class="solr.LowerCaseFilterFactory"/>
        </analyzer>
    </fieldType>

    <!-- charFilter + WhitespaceTokenizer  -->
    <!--
    <fieldType name="text_char_norm" class="solr.TextField" positionIncrementGap="100" >
      <analyzer>
        <charFilter class="solr.MappingCharFilterFactory" mapping="mapping-ISOLatin1Accent.txt"/>
        <tokenizer class="solr.WhitespaceTokenizerFactory"/>
      </analyzer>
    </fieldType>
    -->

    <!-- This is an example of using the KeywordTokenizer along
         With various TokenFilterFactories to produce a sortable field
         that does not include some properties of the source text
      -->
    <fieldType name="alphaOnlySort" class="solr.TextField" sortMissingLast="true" omitNorms="true">
        <analyzer>
            <!-- KeywordTokenizer does no actual tokenizing, so the entire
                 input string is preserved as a single token
              -->
            <tokenizer class="solr.KeywordTokenizerFactory"/>
            <!-- The LowerCase TokenFilter does what you expect, which can be
                 when you want your sorting to be case insensitive
              -->
            <filter class="solr.LowerCaseFilterFactory" />
            <!-- The TrimFilter removes any leading or trailing whitespace -->
            <filter class="solr.TrimFilterFactory" />
            <!-- The PatternReplaceFilter gives you the flexibility to use
                 Java Regular expression to replace any sequence of characters
                 matching a pattern with an arbitrary replacement string,
                 which may include back references to portions of the original
                 string matched by the pattern.

                 See the Java Regular Expression documentation for more
                 information on pattern and replacement string syntax.

                 http://docs.oracle.com/javase/7/docs/api/java/util/regex/package-summary.html
              -->
            <filter class="solr.PatternReplaceFilterFactory"
                    pattern="([^a-z])" replacement="" replace="all"
                    />
        </analyzer>
    </fieldType>

    <fieldtype name="phonetic" stored="false" indexed="true" class="solr.TextField" >
        <analyzer>
            <tokenizer class="solr.StandardTokenizerFactory"/>
            <filter class="solr.DoubleMetaphoneFilterFactory" inject="false"/>
        </analyzer>
    </fieldtype>

    <fieldtype name="payloads" stored="false" indexed="true" class="solr.TextField" >
        <analyzer>
            <tokenizer class="solr.WhitespaceTokenizerFactory"/>
            <!--
            The DelimitedPayloadTokenFilter can put payloads on tokens... for example,
            a token of "foo|1.4"  would be indexed as "foo" with a payload of 1.4f
            Attributes of the DelimitedPayloadTokenFilterFactory :
             "delimiter" - a one character delimiter. Default is | (pipe)
         "encoder" - how to encode the following value into a playload
            float -> org.apache.lucene.analysis.payloads.FloatEncoder,
            integer -> o.a.l.a.p.IntegerEncoder
            identity -> o.a.l.a.p.IdentityEncoder
                Fully Qualified class name implementing PayloadEncoder, Encoder must have a no arg constructor.
             -->
            <filter class="solr.DelimitedPayloadTokenFilterFactory" encoder="float"/>
        </analyzer>
    </fieldtype>

    <!-- lowercases the entire field value, keeping it as a single token.  -->
    <fieldType name="lowercase" class="solr.TextField" positionIncrementGap="100">
        <analyzer>
            <tokenizer class="solr.KeywordTokenizerFactory"/>
            <filter class="solr.LowerCaseFilterFactory" />
        </analyzer>
    </fieldType>

    <!--
      Example of using PathHierarchyTokenizerFactory at index time, so
      queries for paths match documents at that path, or in descendent paths
    -->
    <fieldType name="descendent_path" class="solr.TextField">
        <analyzer type="index">
            <tokenizer class="solr.PathHierarchyTokenizerFactory" delimiter="/" />
        </analyzer>
        <analyzer type="query">
            <tokenizer class="solr.KeywordTokenizerFactory" />
        </analyzer>
    </fieldType>
    <!--
      Example of using PathHierarchyTokenizerFactory at query time, so
      queries for paths match documents at that path, or in ancestor paths
    -->
    <fieldType name="ancestor_path" class="solr.TextField">
        <analyzer type="index">
            <tokenizer class="solr.KeywordTokenizerFactory" />
        </analyzer>
        <analyzer type="query">
            <tokenizer class="solr.PathHierarchyTokenizerFactory" delimiter="/" />
        </analyzer>
    </fieldType>

    <!-- since fields of this type are by default not stored or indexed,
         any data added to them will be ignored outright.  -->
    <fieldtype name="ignored" stored="false" indexed="false" multiValued="true" class="solr.StrField" />

    <!-- This point type indexes the coordinates as separate fields (subFields)
      If subFieldType is defined, it references a type, and a dynamic field
      definition is created matching *___<typename>.  Alternately, if
      subFieldSuffix is defined, that is used to create the subFields.
      Example: if subFieldType="double", then the coordinates would be
        indexed in fields myloc_0___double,myloc_1___double.
      Example: if subFieldSuffix="_d" then the coordinates would be indexed
        in fields myloc_0_d,myloc_1_d
      The subFields are an implementation detail of the fieldType, and end
      users normally should not need to know about them.
     -->
    <fieldType name="point" class="solr.PointType" dimension="2" subFieldSuffix="_d"/>

    <!-- A specialized field for geospatial search. If indexed, this fieldType must not be multivalued. -->
    <fieldType name="location" class="solr.LatLonType" subFieldSuffix="_coordinate"/>

    <!-- An alternative geospatial field type new to Solr 4.  It supports multiValued and polygon shapes.
      For more information about this and other Spatial fields new to Solr 4, see:
      http://wiki.apache.org/solr/SolrAdaptersForLuceneSpatial4
    -->
    <fieldType name="location_rpt" class="solr.SpatialRecursivePrefixTreeFieldType"
               geo="true" distErrPct="0.025" maxDistErr="0.000009" units="degrees" />

    <fieldType name="currency" class="solr.CurrencyField" precisionStep="8" defaultCurrency="USD" currencyConfig="currency.xml" />

</schema>
