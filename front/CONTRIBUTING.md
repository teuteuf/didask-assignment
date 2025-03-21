# Our React Coding Standards

In order to keep our React application maintainable, we agreed on some rules and standards to make sure we have an unified code base.

These rules will be used as a guide for development and code review.

This document is always open to update, if you'd like to suggest an addition or modification, please open a PR and share it on the Slack #dev chanel.

## Resources

Here are interesting resources about good practices for React development:
- https://github.com/mithi/react-philosophies
- https://github.com/alan2207/bulletproof-react/

## Our Rules

### Keep components small and simple ([ref](https://github.com/mithi/react-philosophies?tab=readme-ov-file#-23-keep-your-components-small-and-simple))

<details>
    <summary>❌ View a not-so-good solution</summary>

```tsx
type ShopCategoryTileProps = {
  isBooked: boolean
  icon: ReactNode
  label: string
  componentInsideModal?: ReactNode
  items?: {name: string, quantity: number}[]
}

const ShopCategoryTile = ({
  icon,
  label,
  items
  componentInsideModal,
}: ShopCategoryTileProps ) => {
  const [openDialog, setOpenDialog] = useState(false)
  const [hover, setHover] = useState(false)
  const disabled = !items || items.length  === 0
  return (
    <>
      <Tooltip title="Not Available" show={disabled}>
        <StyledButton
          className={disabled ? "grey" : isBooked ? "green" : "red" }
          disabled={disabled}
          onClick={() => disabled ? null : setOpenDialog(true) }
          onMouseEnter={() => disabled ? null : setHover(true)}
          onMouseLeave={() => disabled ? null : setHover(false)}
        >
          {icon}
          <StyledLabel>{label}<StyledLabel/>
          {!disabled && isBooked && <FaCheckCircle/>}
          {!disabled && hover && <WavingHand />}
        </StyledButton>
      </Tooltip>
      {componentInsideModal &&
        <Dialog open={openDialog} onClose={() => setOpenDialog(false)}>
          {componentInsideModal}
        </Dialog>
      }
    </>
  )
}
```

</details>
          
<details>
    <summary>✅ View a "better" solution</summary>

```tsx
// split into two smaller components!

const DisabledShopCategoryTile = ({ icon, label }: { icon: ReactNode, label: string }) => {
  return (
    <Tooltip title="Not available">
      <StyledButton disabled={true} className="grey">
        {icon}
        <StyledLabel>{label}<StyledLabel/>
      </Button>
    </Tooltip>
  )
}

type ShopCategoryTileProps = {
  icon: ReactNode
  label: string
  isBooked: boolean
  componentInsideModal: ReactNode
}

const ShopCategoryTile = ({
  icon,
  label,
  isBooked,
  componentInsideModal,
}: ShopCategoryTileProps ) => {
  const [openDialog, setOpenDialog] = useState(false)
  const [hover, setHover] = useState(false)

  return (
    <>
      <StyledButton
        disabled={false}
        className={isBooked ? "green" : "red"}
        onClick={() => setOpenDialog(true) }
        onMouseEnter={() => setHover(true)}
        onMouseLeave={() => setHover(false)}
      >
        {icon}
        <StyledLabel>{label}<StyledLabel/>
        {isBooked && <FaCheckCircle/>}
        {hover && <WavingHand />}
      </StyledButton>
      {openDialog &&
        <Dialog onClose={() => setOpenDialog(false)}>
          {componentInsideModal}
        </Dialog>
      }}
    </>
  )
}
```

</details>

### Extract business logic from UI to custom hooks


<details>
    <summary>❌ View a not-so-good solution</summary>

```tsx
// OnlineGame.tsx
export const OnlineGame = ({ gameId }: { gameId: string}) => {
  const { session } = useContext(SessionContext);

  const queryClient = useQueryClient();
  const gameData = useQuery<GameData | null>({
    ...
  });

  useEffect(() => {
    const unsubscribe = subscribeToGameUpdates({
      gameId,
      onGameUpdate: (gameData) => {
        ...
      },
    });

    return () => {
      unsubscribe();
    };
  }, [gameId, queryClient, session]);

  if (!gameData) {
    return (
      <LoadingSpinner>
    );
  }

  return <Game gameData={gameData} />
}
```

</details>
          
<details>
    <summary>✅ View a "better" solution</summary>

```tsx
// useGameData.ts
export const useGameData = (gameId: string) => {
  const { session } = useContext(SessionContext);

  const queryClient = useQueryClient();
  const result = useQuery<GameData | null>({
    ...
  });

  useEffect(() => {
    const unsubscribe = subscribeToGameUpdates({
      gameId,
      onGameUpdate: (gameData) => {
        ...
      },
    });

    return () => {
      unsubscribe();
    };
  }, [gameId, queryClient, session]);

  return result;
};

// OnlineGame.tsx
export const OnlineGame = ({ gameId }: { gameId: string}) => {
  const gameData = useGameData(gameId)

  if (!gameData) {
    return (
      <LoadingSpinner>
    );
  }

  return <Game gameData={gameData} />
}
```

</details>