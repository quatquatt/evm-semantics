KEVM Optimizations
==================

These optimizations work on the LLVM and Haskell backend and are generated by the script `./optimizer/optimizations.sh`.

```k
requires "evm.md"

module EVM-OPTIMIZATIONS
  imports EVM

  rule
  [optimized.pushzero]:
    <kevm>
      <k>
        ( #next[ PUSHZERO ] => .K ) ...
      </k>
      <schedule>
        SCHED
      </schedule>
      <useGas>
        USEGAS
      </useGas>
      <ethereum>
        <evm>
          <callState>
            <wordStack>
              ( WS => 0 : WS )
            </wordStack>
            <pc>
              ( PCOUNT => ( PCOUNT +Int 1 ) )
            </pc>
            <gas>
              ( GAVAIL => ( #if USEGAS #then ( GAVAIL -Gas Gbase < SCHED > ) #else GAVAIL #fi ) )
            </gas>
            ...
          </callState>
          ...
        </evm>
        ...
      </ethereum>
      ...
    </kevm>
    requires ( #if USEGAS #then Gbase < SCHED > <=Gas GAVAIL #else true #fi )
     andBool ( #sizeWordStack( 0 : WS ) <=Int 1024 )
     [priority(40)]

  rule
  [optimized.push]:
    <kevm>
      <k>
        ( #next[ PUSH(N) ] => .K ) ...
      </k>
      <schedule>
        SCHED
      </schedule>
      <useGas>
        USEGAS
      </useGas>
      <ethereum>
        <evm>
          <callState>
            <program>
              PGM
            </program>
            <wordStack>
              ( WS => #asWord( #range(PGM, PCOUNT +Int 1, N) ) : WS )
            </wordStack>
            <pc>
              ( PCOUNT => ( ( PCOUNT +Int N ) +Int 1 ) )
            </pc>
            <gas>
              ( GAVAIL => #if USEGAS #then ( GAVAIL -Gas Gverylow < SCHED > ) #else GAVAIL #fi )
            </gas>
            ...
          </callState>
          ...
        </evm>
        ...
      </ethereum>
      ...
    </kevm>
    requires ( #if USEGAS #then Gverylow < SCHED > <=Gas GAVAIL #else true #fi )
     andBool ( #sizeWordStack( #asWord( #range(PGM, PCOUNT +Int 1, N) ) : WS ) <=Int 1024 )
     [priority(40)]

  rule
  [optimized.dup]:
    <kevm>
      <k>
        ( #next[ DUP(N) ] => .K ) ...
      </k>
      <schedule>
        SCHED
      </schedule>
      <useGas>
        USEGAS
      </useGas>
      <ethereum>
        <evm>
          <callState>
            <wordStack>
              ( WS => WS [ ( N +Int -1 ) ] : WS )
            </wordStack>
            <pc>
              ( PCOUNT => ( PCOUNT +Int 1 ) )
            </pc>
            <gas>
              ( GAVAIL => #if USEGAS #then ( GAVAIL -Gas Gverylow < SCHED > ) #else GAVAIL #fi )
            </gas>
            ...
          </callState>
          ...
        </evm>
        ...
      </ethereum>
      ...
    </kevm>
    requires #stackNeeded(DUP(N)) <=Int #sizeWordStack(WS)
     andBool ( #if USEGAS #then Gverylow < SCHED > <=Gas GAVAIL #else true #fi )
     andBool ( #sizeWordStack( WS [ ( N +Int -1 ) ] : WS ) <=Int 1024 )
     [priority(40)]

  rule
  [optimized.swap]:
    <kevm>
      <k>
        ( #next[ SWAP(N) ] => .K ) ...
      </k>
      <schedule>
        SCHED
      </schedule>
      <useGas>
        USEGAS
      </useGas>
      <ethereum>
        <evm>
          <callState>
            <wordStack>
              ( W0 : WS => WS [ ( N +Int -1 ) ] : ( WS [ ( N +Int -1 ) := W0 ] ) )
            </wordStack>
            <pc>
              ( PCOUNT => ( PCOUNT +Int 1 ) )
            </pc>
            <gas>
              ( GAVAIL => #if USEGAS #then ( GAVAIL -Gas Gverylow < SCHED > ) #else GAVAIL #fi )
            </gas>
            ...
          </callState>
          ...
        </evm>
        ...
      </ethereum>
      ...
    </kevm>
    requires #stackNeeded(SWAP(N)) <=Int #sizeWordStack(W0 : WS)
     andBool ( #if USEGAS #then Gverylow < SCHED > <=Gas GAVAIL #else true #fi )
     andBool ( #sizeWordStack( WS [ ( N +Int -1 ) ] : ( WS [ ( N +Int -1 ) := W0 ] ) ) <=Int 1024 )
     [priority(40)]

  rule
  [optimized.add]:
    <kevm>
      <k>
        ( #next[ ADD ] => .K ) ...
      </k>
      <schedule>
        SCHED
      </schedule>
      <useGas>
        USEGAS
      </useGas>
      <ethereum>
        <evm>
          <callState>
            <wordStack>
              ( W0 : W1 : WS => chop( ( W0 +Int W1 ) ) : WS )
            </wordStack>
            <pc>
              ( PCOUNT => ( PCOUNT +Int 1 ) )
            </pc>
            <gas>
              ( GAVAIL => #if USEGAS #then ( GAVAIL -Gas Gverylow < SCHED > ) #else GAVAIL #fi )
            </gas>
            ...
          </callState>
          ...
        </evm>
        ...
      </ethereum>
      ...
    </kevm>
    requires ( #if USEGAS #then Gverylow < SCHED > <=Gas GAVAIL #else true #fi )
     andBool ( #sizeWordStack( chop( ( W0 +Int W1 ) ) : WS ) <=Int 1024 )
     [priority(40)]

  rule
  [optimized.sub]:
    <kevm>
      <k>
        ( #next[ SUB ] => .K ) ...
      </k>
      <schedule>
        SCHED
      </schedule>
      <useGas>
        USEGAS
      </useGas>
      <ethereum>
        <evm>
          <callState>
            <wordStack>
              ( W0 : W1 : WS => chop( ( W0 -Int W1 ) ) : WS )
            </wordStack>
            <pc>
              ( PCOUNT => ( PCOUNT +Int 1 ) )
            </pc>
            <gas>
              ( GAVAIL => #if USEGAS #then ( GAVAIL -Gas Gverylow < SCHED > ) #else GAVAIL #fi )
            </gas>
            ...
          </callState>
          ...
        </evm>
        ...
      </ethereum>
      ...
    </kevm>
    requires ( #if USEGAS #then Gverylow < SCHED > <=Gas GAVAIL #else true #fi )
     andBool ( #sizeWordStack( chop( ( W0 -Int W1 ) ) : WS ) <=Int 1024 )
     [priority(40)]

  rule
  [optimized.and]:
    <kevm>
      <k>
        ( #next[ AND ] => .K ) ...
      </k>
      <schedule>
        SCHED
      </schedule>
      <useGas>
        USEGAS
      </useGas>
      <ethereum>
        <evm>
          <callState>
            <wordStack>
              ( W0 : W1 : WS => W0 &Int W1 : WS )
            </wordStack>
            <pc>
              ( PCOUNT => ( PCOUNT +Int 1 ) )
            </pc>
            <gas>
              ( GAVAIL => #if USEGAS #then ( GAVAIL -Gas Gverylow < SCHED > ) #else GAVAIL #fi )
            </gas>
            ...
          </callState>
          ...
        </evm>
        ...
      </ethereum>
      ...
    </kevm>
    requires ( #if USEGAS #then Gverylow < SCHED > <=Gas GAVAIL #else true #fi )
     andBool ( #sizeWordStack( W0 &Int W1 : WS ) <=Int 1024 )
     [priority(40)]

  rule
  [optimized.lt]:
    <kevm>
      <k>
        ( #next[ LT ] => .K ) ...
      </k>
      <schedule>
        SCHED
      </schedule>
      <useGas>
        USEGAS
      </useGas>
      <ethereum>
        <evm>
          <callState>
            <wordStack>
              ( W0 : W1 : WS => bool2Word( W0 <Int W1 ) : WS )
            </wordStack>
            <pc>
              ( PCOUNT => ( PCOUNT +Int 1 ) )
            </pc>
            <gas>
              ( GAVAIL => #if USEGAS #then ( GAVAIL -Gas Gverylow < SCHED > ) #else GAVAIL #fi )
            </gas>
            ...
          </callState>
          ...
        </evm>
        ...
      </ethereum>
      ...
    </kevm>
    requires ( #if USEGAS #then Gverylow < SCHED > <=Gas GAVAIL #else true #fi )
     andBool ( #sizeWordStack( bool2Word( W0 <Int W1 ) : WS ) <=Int 1024 )
     [priority(40)]

  rule
  [optimized.gt]:
    <kevm>
      <k>
        ( #next[ GT ] => .K ) ...
      </k>
      <schedule>
        SCHED
      </schedule>
      <useGas>
        USEGAS
      </useGas>
      <ethereum>
        <evm>
          <callState>
            <wordStack>
              ( W0 : W1 : WS => bool2Word( W1 <Int W0 ) : WS )
            </wordStack>
            <pc>
              ( PCOUNT => ( PCOUNT +Int 1 ) )
            </pc>
            <gas>
              ( GAVAIL => #if USEGAS #then ( GAVAIL -Gas Gverylow < SCHED > ) #else GAVAIL #fi )
            </gas>
            ...
          </callState>
          ...
        </evm>
        ...
      </ethereum>
      ...
    </kevm>
    requires ( #if USEGAS #then Gverylow < SCHED > <=Gas GAVAIL #else true #fi )
     andBool ( #sizeWordStack( bool2Word( W1 <Int W0 ) : WS ) <=Int 1024 )
     [priority(40)]


// {OPTIMIZATIONS}


endmodule
```
