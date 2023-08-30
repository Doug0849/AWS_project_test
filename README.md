V2 修改過啦！

1. 必須在父層加上以下code
```typescript
const router = useRouter();
const [loading, setLoading] = useState(false);

useEffect(() => {
    const handleStart = (url) => (url !== router.asPath) && setLoading(true);
    const handleComplete = (url) => (url === router.asPath) && setLoading(false);

    router.events.on('routeChangeStart', handleStart)
    router.events.on('routeChangeComplete', handleComplete)
    router.events.on('routeChangeError', handleComplete)

    return () => {
        router.events.off('routeChangeStart', handleStart)
        router.events.off('routeChangeComplete', handleComplete)
        router.events.off('routeChangeError', handleComplete)
    }
  })
```

2. 接著在return的時後，以loading作為判斷選擇render
```javascript
return (
    <> 
      <Box as="main" w="100vw" h="100vh" bgColor="color.1" position="relative">
          { loading ? (<Loading/>) : ( 
            <>
              <Navbar urlPath={urlPath} />
              {children}
            </>
          )}
      </Box>
    </>
```

3. 在 / 目錄下的需要帶入 export getSeverSidePops() 以Nero案為例只有在BaseLayout及Jumbotron 兩個檔案有加入
```typescript
export async function getServerSideProps() {

  await new Promise((resolve) => {
    setTimeout(resolve,1000)
  })

  return {
    props: {},
  }
}
```

4. 若要在一進入網站就先掛載loading，最後改寫為以下，可以將loading page轉移到baseLayout以上作為父層(nero案子為例)
```typescript
import {
  Box,
  useDisclosure,
  useBreakpoint
} from "@chakra-ui/react";
import Link from "next/link";
import Navbar from "./Navbar";
// import { useTranslation } from "next-i18next";
import React, { Suspense, useEffect, useState } from "react";
import router, { useRouter } from "next/router";
import Loading from "../Loading copy 2";

export default function BaseLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const { isOpen, onToggle, onClose } = useDisclosure();
  const breakpoint = useBreakpoint();
  const isMobile = breakpoint === "md" || breakpoint === "sm";
  const urlPath = useRouter().pathname;

  const [isLoading, setIsLoading] = useState(true);

    useEffect(() => {
      const handleStart = (url:string) => (url !== router.asPath) && setIsLoading(true);
      setTimeout (() =>{setIsLoading (false)} ,3000) // 增加一開始進入首頁的loading
      // 以下為轉換頁面，變動router的時候，也同樣會跑loading。
      const handleComplete = (url:string) => (url === router.asPath) && setTimeout(()=>{setIsLoading(false)},3000)
      // 要跟上面一樣久

      router.events.on('beforeHistoryChange',handleStart)
      router.events.on('routeChangeComplete', handleComplete)
      router.events.on('routeChangeStart', handleStart)
      router.events.on('routeChangeComplete', handleComplete)
      router.events.on('routeChangeError', handleComplete)

      return () => {
          router.events.off('beforeHistoryChange',handleStart)
          router.events.off('routeChangeStart', handleStart)
          router.events.off('routeChangeComplete', handleComplete)
          router.events.off('routeChangeError', handleComplete)
      }
    })
  
  return (
    <> 
      { isLoading && (<Loading/>) } // 如果是true的話就加頁面載。一開始就會先loading。
      <Box as="main" w="100vw" h="100vh" bgColor="color.1" position="relative">
          <Navbar urlPath={urlPath} />
          {children}
      </Box>
    </>
  )}

// export async function getServerSideProps() {

//   await new Promise((resolve) => {
//     setTimeout(resolve,1000)
//   })

//   return {
//     props: {},
//   }
// }
```


5. 完整程式碼

BaseLayout.tsx
```typescript
import {
  Box,
  useDisclosure,
  useBreakpoint
} from "@chakra-ui/react";
import Link from "next/link";
import Navbar from "./Navbar";
import { useTranslation } from "next-i18next";
import React, { Suspense, useEffect, useState } from "react";
import CustomDailyGoalButton from "../buttons/CustomDailyGoalButton";
import { useRouter } from "next/router";
import Loading from "../Loading";

export default function BaseLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const { isOpen, onToggle, onClose } = useDisclosure();
  const { t } = useTranslation(["navbar", "social"]);
  // const { t: t_social } = useTranslation();
  // console.log('2')
  const breakpoint = useBreakpoint();
  const isMobile = breakpoint === "md" || breakpoint === "sm";
  const urlPath = useRouter().pathname;

  const router = useRouter();
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    const handleStart = (url) => (url !== router.asPath) && setLoading(true);
    const handleComplete = (url) => (url === router.asPath) && setLoading(false);

    router.events.on('routeChangeStart', handleStart)
    router.events.on('routeChangeComplete', handleComplete)
    router.events.on('routeChangeError', handleComplete)

    return () => {
        router.events.off('routeChangeStart', handleStart)
        router.events.off('routeChangeComplete', handleComplete)
        router.events.off('routeChangeError', handleComplete)
    }
  })
  
  return (
    <> 
      <Box as="main" w="100vw" h="100vh" bgColor="color.1" position="relative">
          { loading ? (<Loading/>) : ( 
            <>
              <Navbar urlPath={urlPath} />
              {children}
            </>
          )}
      </Box>
    </>
  )}
  
  
```

loading.tsx
```typescript
  import { Box, Flex, Img, Spinner, Text } from "@chakra-ui/react"
  import LoadingBg from "../public/images/loadingBg"
  import { useEffect, useState } from "react"
  import React from "react"

  export default function Loading() {
      return (
        <>
          <Flex className="loading"
            h="2000px"
            w="100vw"
            position="absolute"
            justify="center"
            align="start"
            textStyle="text01"
            textAlign="center"
            bgColor="color.0"
            zIndex="10"
            color="color.3"
            overflow="hidden"
            >
            <LoadingBg />
            <Flex className="LoadingWrapper"
              w="100vw"
              h="100vh"
              direction="column"
              justify="center"
              align="center"
            >
              <Box
                mt="10%"
                mb="-20px"
                w={{ sm:"242px", md:"280px", lg:"322px"}}
                h={{ sm:"415px", md:"481px", lg:"553px"}}
                bgImage="url('./images/loading.png')"
                bgPosition="center"
                bgSize="contain"
                bgRepeat="no-repeat"
                zIndex="1"
              />
              <Box
                mb="30px"
              >
                <Text textStyle={{ sm: "text03", md: "text03", lg: "text01"}}>
                  Loading...
                </Text>
              </Box>
              <Spinner 
                thickness='4px'
                speed='0.65s'
                emptyColor='color.4'
                size={{ sm:"lg", md:"lg", ls:"xl"}}
              />
            </Flex>
          </Flex> 
        </>
        )
  }
```
