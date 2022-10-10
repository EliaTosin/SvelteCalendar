<script>
    var a = "A"
 
    function testThis(){
    // global object (window)
    console.log("1: "+this);
    console.log("2: "+this.a);
    
    function test2(){
    console.log("3: "+this);
    }
    }
    
    testThis();
    
    // cambiamo this
    testThis.call({a: 'AAA'})
</script>