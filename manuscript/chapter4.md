## Formateando Dinero



La interacción con el usuario en la mayoría de sistemas es una parte fundamental, y asi es necesario pensar en la forma de presentar la información para el usuario además de la interacción con el software. Dinero es una parte importante de esos softwares, asi es importante exhibir el gasto total por un servicio, importe total de productos que el usuario desea comprar. Sin hablar en la forma de interacción, por ejemplo, informar el dinero que será transferido por otra cuenta via banca por internet. Para trabajar con el formateo de un ```MonetaryAmount``` existe la interface ```MonetaryAmountFormat``` que básicamente muestra ```MonetaryAmount``` como ```String``` y recupera un ```MonetaryAmount``` a partir de un ```String```. 


```java
public interface MonetaryAmountFormat extends MonetaryQuery<String>{

   AmountFormatContext getContext();

   default String format(MonetaryAmount amount){...}

   void print(Appendable appendable, MonetaryAmount amount) throws IOException;

   MonetaryAmount parse(CharSequence text) throws MonetaryParseException;

}
```

Um ejemplo simple es **toString** y el parseo dentro de las implementaciones de ```MonetaryAmount``` dentro de Moneta.


```java
public class ToStrimgExample {

    public static void main(String[] args) {
        CurrencyUnit currency = Monetary.getCurrency(Locale.US);
        MonetaryAmount money = Money.of(10, currency);
        MonetaryAmount money2 = FastMoney.of(20, currency);
        MonetaryAmount money3 = RoundedMoney.of(30, currency, MonetaryOperators.rounding());
        String text = money.toString();//USD 10
        String text2 = money2.toString();//USD 20
        String text3 = money3.toString();//USD 30
        MonetaryAmount result = Money.parse(text);
        MonetaryAmount result2 = FastMoney.parse(text2);
        MonetaryAmount result3 = RoundedMoney.parse(text3);

    }
}
```
### La clase MonetaryAmountFormat

La implementación de referencia tiene dos formas de crear formato para Dinero. La primera opción es con la clase  ```MonetaryFormats``` con esta es posible crear un formateador a partir de un query builder o utilizando solo por ```Locale```. Además del parseo sin parametros en las implementaciones de ```MonetaryAmount```, también existe el metodo parse que acepta ```MonetaryAmountFormat```.


```java
public class MonetaryFormatsExample {


    public static void main(String[] args) {
        CurrencyUnit currency = Monetary.getCurrency("EUR");
        MonetaryAmount money = Money.of(12, currency);

        MonetaryAmountFormat format =
                MonetaryFormats.getAmountFormat(Locale.US);

        String resultText = format.format(money);//EUR 12
        MonetaryAmount monetaryAmount = format.parse(resultText);
        MonetaryAmount result2 = Money.parse(resultText, format);

    }
}
```



```java
public class MonetaryFormatsExampleQuery {

    public static void main(String[] args) {
        CurrencyUnit currency = Monetary.getCurrency("USD");
        MonetaryAmount money = Money.of(12, currency);

        MonetaryAmountFormat format = MonetaryFormats
                        .getAmountFormat(AmountFormatQueryBuilder.of(Locale.US).set(CurrencyStyle.SYMBOL).build());


        String resultText = format.format(money);//$12.00
    }
}
```

    ### La clase MonetaryAmountFormatSymbols

Existe también la classe ```MonetaryAmountDecimalFormatBuilder```, que de forma semejante la clase ```DecimalFormat``` con la clase ```Number```, tiene el objetivo de realizar formateadores de dinero a partir del as configuraciones de símbolos, monedas, cantidad mínima y máxima de dígitos antes y despues la coma decimal, etc.


```java
public class MonetaryAmountDecimalFormatBuilderExample {

    public static void main(String[] args) {
        MonetaryAmountFormat defaultFormat = MonetaryAmountDecimalFormatBuilder.newInstance().build();
        MonetaryAmountFormat patternFormat = MonetaryAmountDecimalFormatBuilder.of("¤ ###,###.00").build();
        MonetaryAmountFormat localeFormat = MonetaryAmountDecimalFormatBuilder.of(Locale.US).build();
        
        CurrencyUnit currency = Monetary.getCurrency("BRL");
        MonetaryAmount money = Money.of(12, currency);
        String format = defaultFormat.format(money);//$12.00
        MonetaryAmount moneyParsed = Money.parse(format, defaultFormat);//or using defafult.parse(format);

    }
}

```


También es posible definir cual implementación será utilizada en la serialización del objeto. Para eso existe la clase funcional ```MonetaryAmountProducer```, con esta es posible definir su propia implementación a partir del número y de la moneda. Moneta por estándar ya viene con tres implementaciones:


* ```FastMoneyProducer``` productor de ```MonetaryAmount``` con la implementación ```FastMoney```.
* ```MoneyProducer``` productor de ```MonetaryAmount``` con la implementación ```Money```.
* ```RoundedMoneyProducer``` productor de ```MonetaryAmount``` con la implementación ```RoundedMoney```, en esta es posible pasar un ```MonetaryOperator``` que será utilizado en la construcción de esa implementación asi no sea informado un operador de redondeo será utilizado ```MonetaryOperators.rounding()```. Recordar que ```MonetaryOperator``` dentro de la implementación ```RoundedMoney``` será utilizado como agente de redondedo, osea, para cada operación ese operador será llamado.



```java
public class MonetaryAmountDecimalFormatBuilderExample2 {

    public static void main(String[] args) {
        CurrencyUnit currency = Monetary.getCurrency("BRL");
        MonetaryAmount money = Money.of(12, currency);
        
        MonetaryAmountFormat formater = MonetaryAmountDecimalFormatBuilder.of(new Locale("pt", "BR")).
                withCurrencyUnit(currency).withProducer(new MoneyProducer()).build();
        
        
        String format = formater.format(money);//R$ 12,00
        MonetaryAmount moneyParsed = Money.parse(format, formater);//or using defafult.parse(format);

    }
}
```

También es posible pasar un ```String``` como patrón para el formateo, ese ```String``` sigue el mismo standard de la clase ```DecimalFormat```.


```java
public class MonetaryAmountDecimalFormatBuilderExample3 {

    public static void main(String[] args) {
        MonetaryAmountFormat patternFormat = MonetaryAmountDecimalFormatBuilder.of("¤ ###,###.00").build();
        
        CurrencyUnit currency = Monetary.getCurrency("BRL");
        MonetaryAmount money = Money.of(12, currency);
        String format = patternFormat.format(money);//$ 12.00
        MonetaryAmount moneyParsed = Money.parse(format, patternFormat);//or using defafult.parse(format);

    }
}

```
